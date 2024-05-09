# Implementation of Multisig Signatures
A (t, n) threshold signature scheme (TSS) is a cryptographic protocol that enables a group of n participants to collectively sign documents or transactions. In this scheme, any subset of t + 1 or more participants can generate a valid signature, while subsets of t or fewer participants cannot. The primary goal of this scheme is to enhance security and trust in distributed systems by preventing any single party from unilaterally producing a signature, thereby thwarting unauthorized actions. TSS achieves this by distributing control over cryptographic keys among multiple parties, ensuring that the key never exists in a single location.

There exist several signature schemes that can be extended to a threshold version, such as ECDSA, Schnorr, and BLS. We have opted for the ECDSA version of TSS due to its native support in Bitcoin and EVM ecosystems. Our implementation is based on the [tss-lib](https://github.com/bnb-chain/tss-lib) library.

## tss-lib Project
This project implements a multi-party {t, n}-threshold ECDSA (Elliptic Curve Digital Signature Algorithm) and EdDSA (Edwards-curve Digital Signature Algorithm) using a similar methodology.

The library encompasses three main protocols:

- Key Generation: This protocol generates secret shares without the need for a trusted dealer ("keygen").
- Signing: Involves utilizing the secret shares to create a signature ("signing").
- Dynamic Groups: Allows for changing the group of participants while preserving the secrecy of the shared key ("resharing").

## @rosen-bridge/tss-api
The @rosen-bridge/tss-api project facilitates key generation and signing operations for eddsa and ecdsa protocols within a threshold signature setup. It is built on top of Binance's TSS-lib.

For message passing, the `ts-guard-service` project should be executed, with its port specified as the `guardUrl` argument.

We have developed the [tss-api](https://github.com/rosen-bridge/tss-api) implementation over tss-lib, offering additional APIs for key generation and signing transactions.

The `Dialer` component is a messaging module implemented in the `guard-service` utilizing [libp2p](https://github.com/libp2p/js-libp2p) for efficient message passing between tss-api parties.

Below is a breakdown of tss-api's modules:
- ECDSA keygen module for EVM and Bitcoin networks can be found [here](https://github.com/rosen-bridge/tss-api/tree/refactor-feb/app/keygen/ecdsa).
- ECDSA sign module for EVM and Bitcoin networks is available [here](https://github.com/rosen-bridge/tss-api/tree/refactor-feb/app/sign/ecdsa):
    - The signing process involves child key derivation, where a new path is derived per network with a distinct chain code based on the BIP-32 specification. Our implementations are detailed in [[ref-1](https://github.com/rosen-bridge/tss-api/blob/bd7a1b2ffdca6ad14a4b5730f2a3a491e7b41c49/app/sign/ecdsa/ecdsa.go#L206-L234), [ref-2](https://github.com/rosen-bridge/utils/blob/dev/packages/tss/lib/tss/EcdsaSigner.ts)].

### Tests

The very first manual signature test has been done for [this](https://www.bscscan.com/address/0x2ac3fba3d67e4629e118202a5c459f3711d1f9e0) multisig address on Binance Smart Chain, including a number of transactions like [this transaction](https://www.bscscan.com/tx/0x2bbc43d23f764a2d5c5ba3f4bd6fc89acef242234ab6089d78015692d7907002).


Tests have been conducted in a real environment. The first transaction with the derivation path [12, 209, 3] and chain code `Bitcoin Seed` using tss-api over Bitcoin can be viewed [here](https://blockstream.info/tx/b75c46f83cd61da124c2c7817c20e9c094e7ae85ad76c61842d6caaea523236d?expand).

We are utilizing a TSS party with a minimum required signature count of 6 out of 10. The child ECDSA public key for this party is `03e01839d46f3df23cfb457e927efe4f764519861afc7ef8d93407b6c4cb164f22`. Here are all the shared root public keys for the guards:
```yaml
- 02a561553a9930c50dfd8175811eb514d5260a73bbe1c3ac832c73891d162130f2
- 023d42fb9109678208ee04a124da8fc54431afe2630c5c151be73a7dc73995c961
- 036a4ff444e4786547f72292c811217e8be71ebdd7838432ac095d32829dbd3854
- 029cef851abe159c486e849d6956c559f218382f683bf2a013c4ef369a467bf7c8
- 023cac18f6ad56b0f09bf690ba6f786c72b190e918df1ea8e5342320a5c6f1d240
- 0320f578e4c302f8328f8569ff3a2aef51491b2c95b93447d191c84d8428f818ad
- 03841ac95f1230bb789a844c7fc0f599e01163f3d954e9e99a4d6c5a32a79f8c8a
- 02491c16b1b3a494923143624d4dbbcd3927ffe73f1312c5331bf6b4af93c9f5d5
- 0266b6f92e783b364fa50aba2633111475201fd6f9ca247e51e81e8fdd9e76230c
- 0350f91b81bdd8aa78df30bfe8d978e44ff4b407778917e9a99bb1e7495d7c2781
```

The root public keys of the participating guards in this transaction are:
```yaml
- 03841ac95f1230bb789a844c7fc0f599e01163f3d954e9e99a4d6c5a32a79f8c8a
- 023cac18f6ad56b0f09bf690ba6f786c72b190e918df1ea8e5342320a5c6f1d240
- 029cef851abe159c486e849d6956c559f218382f683bf2a013c4ef369a467bf7c8
- 036a4ff444e4786547f72292c811217e8be71ebdd7838432ac095d32829dbd3854
- 0266b6f92e783b364fa50aba2633111475201fd6f9ca247e51e81e8fdd9e76230c
- 02a561553a9930c50dfd8175811eb514d5260a73bbe1c3ac832c73891d162130f2
```

### Test Scenario for Sign Message `9a3e23fa0eb7a13ca92affd4ec762882f9163d33f4ca7153acbcd6f63beab057`
`Note`: The keygen_data file is not included in this gist to maintain confidentiality.

```golang
package signing

import (
	"bytes"
	"crypto/ecdsa"
	"encoding/hex"
	"fmt"
	"math/big"
	"sync/atomic"
	"testing"

	"github.com/btcsuite/btcd/btcec/v2"
	"github.com/ipfs/go-log"
	"github.com/stretchr/testify/assert"

	"github.com/bnb-chain/tss-lib/v2/common"
	"github.com/bnb-chain/tss-lib/v2/ecdsa/keygen"
	"github.com/bnb-chain/tss-lib/v2/test"
	"github.com/bnb-chain/tss-lib/v2/tss"
)

const (
	testParticipants = 10
	testThreshold    = 5 // 5 + 1
)

func setUp(level string) {
	if err := log.SetLogLevel("tss-lib", level); err != nil {
		panic(err)
	}
}

func TestE2EWithHDKeyDerivation(t *testing.T) {
	setUp("info")
	threshold := testThreshold
	g := "Bitcoin Seed"
	chainCode := bytes.NewBufferString(g).Bytes()

	// PHASE: load keygen fixtures
	keys, signPIDs, err := keygen.LoadKeygenTestFixturesRandomSet(testThreshold, testParticipants)
	assert.NoError(t, err, "should load keygen fixtures")
	assert.Equal(t, testThreshold, len(keys))
	assert.Equal(t, testThreshold, len(signPIDs))

	il, extendedChildPk, errorDerivation := derivingPubkeyFromPath(keys[0].ECDSAPub, chainCode, []uint32{12, 209, 3}, btcec.S256())
	assert.NoErrorf(t, errorDerivation, "there should not be an error deriving the child public key")

	keyDerivationDelta := il
	fmt.Sprintln()
	err = UpdatePublicKeyAndAdjustBigXj(keyDerivationDelta, keys, &extendedChildPk.PublicKey, btcec.S256())
	assert.NoErrorf(t, err, "there should not be an error setting the derived keys")

	// PHASE: signing
	// use a shuffled selection of the list of parties for this test
	p2pCtx := tss.NewPeerContext(signPIDs)
	parties := make([]*LocalParty, 0, len(signPIDs))

	errCh := make(chan *tss.Error, len(signPIDs))
	outCh := make(chan tss.Message, len(signPIDs))
	endCh := make(chan *common.SignatureData, len(signPIDs))

	updater := test.SharedPartyUpdater
	msgData, _ := hex.DecodeString("9a3e23fa0eb7a13ca92affd4ec762882f9163d33f4ca7153acbcd6f63beab057")

	// init the parties
	for i := 0; i < len(signPIDs); i++ {
		params := tss.NewParameters(tss.S256(), p2pCtx, signPIDs[i], len(signPIDs), threshold)
		data := new(big.Int).SetBytes(msgData)
		P := NewLocalPartyWithKDD(data, params, keys[i], keyDerivationDelta, outCh, endCh, len(msgData)).(*LocalParty)
		parties = append(parties, P)
		go func(P *LocalParty) {
			if err := P.Start(); err != nil {
				errCh <- err
			}
		}(P)
	}

	var ended int32
signing:
	for {
		select {
		case err := <-errCh:
			common.Logger.Errorf("Error: %s", err)
			assert.FailNow(t, err.Error())
			break signing

		case msg := <-outCh:
			dest := msg.GetTo()
			if dest == nil {
				for _, P := range parties {
					if P.PartyID().Index == msg.GetFrom().Index {
						continue
					}
					go updater(P, msg, errCh)
				}
			} else {
				if dest[0].Index == msg.GetFrom().Index {
					t.Fatalf("party %d tried to send a message to itself (%d)", dest[0].Index, msg.GetFrom().Index)
				}
				go updater(parties[dest[0].Index], msg, errCh)
			}

		case <-endCh:
			atomic.AddInt32(&ended, 1)
			if atomic.LoadInt32(&ended) == int32(len(signPIDs)) {
				t.Logf("Done. Received signature data from %d participants", ended)
				R := parties[0].temp.bigR
				r := parties[0].temp.rx
				fmt.Printf("sign result: R(%s, %s), r=%s\n", R.X().String(), R.Y().String(), r.String())

				modN := common.ModInt(tss.S256().Params().N)

				// BEGIN check s correctness
				sumS := big.NewInt(0)
				for _, p := range parties {
					sumS = modN.Add(sumS, p.temp.si)
				}
				fmt.Printf("S: %s\n", sumS.String())
				// END check s correctness

				// BEGIN ECDSA verify
				pkX, pkY := keys[0].ECDSAPub.X(), keys[0].ECDSAPub.Y()
				pk := ecdsa.PublicKey{
					Curve: tss.EC(),
					X:     pkX,
					Y:     pkY,
				}
				ok := ecdsa.Verify(&pk, msgData, R.X(), sumS)
				assert.True(t, ok, "ecdsa verify must pass")
				t.Log("ECDSA signing test done.")
				// END ECDSA verify

				break signing
			}
		}
	}
}


// Result of the test scenario:
// signature: 5efb5f59c6ecf36c9333ae1cac9243d62040c2769f911b86d42107e35c0b0ff61b8bb3a808ee291b2e7e1656e350550302826a935ecf06d5a78c6e3eb5d2c347
// Message: 9a3e23fa0eb7a13ca92affd4ec762882f9163d33f4ca7153acbcd6f63beab057
// SignatureRecovery: 01
```