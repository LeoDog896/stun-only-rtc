# stun-only-rtc

Cleanup of https://github.com/xem/miniWebRTC.

Serverless* WebRTC. Enables browsers to commincate with each other without the need for a costly server.

* Uses STUN servers -- virtually free and hosted by various companies.

## Simple explanation
* BOB: Create an offer (with ice candidates included)
* BOB -> ALICE: Send the offer
* ALICE -> BOB Receive answer
* Connected!

## Code explanation

```ts
// Your local peer connection
const peerConnection = new RTCPeerConnection(config)

// If this is bob (initializer)
/* ... */ {
  peerConnection.setLocalDescription(await peerConnection.createOffer())

  peerConnection.addEventListener("icecandidate", ({ candidate }) => {
    if (candidate == null) {
      // OFFER from bob. Send this to alice.
      console.log(JSON.stringify(peerConnection.localDescription));
    }
  })

  // Run this function when bob has the answer from alice.
  when("bob receives the answer", async (answer: string) => {
    const answerDesc = new RTCSessionDescription(JSON.parse(answer))
    await peerConnection.setRemoteDescription(answerDesc);
    // CONNECTED!
  });

  // ALICE sent a message to BOB
  dataChannel.addEventListener("message", e => {
    const data = JSON.parse(e.data)
    // You have the message!
    console.log(data.message)
  })
}

// If this is alice (joiner)
/* ... */ {
  when("alice gets their offer from bob", async (offer: string) => {
    const offerDesc = new RTCSessionDescription(JSON.parse(offer))
    await peerConnection.setRemoteDescription(offerDesc)
    peerConnection.setLocalDescription(await peerConnection.createAnswer())
    peerConnection.addEventListener("icecandidate", e => {
      if (e.candidate == null) {
        // ALICE has her answer. Send this to bob.
        console.log(JSON.stringify(peerConnection.localDescription));
      }
    })
  ));

  peerConnection.addEventListener("datachannel", ({ channel }) => {
    // CONNECTED!

    // BOB sent a message to ALICE
    channel.addEventListener("message", e => {
      const data = JSON.parse(e.data)
      // You have the message!
      console.log(data.message)
    })
  })
}
```