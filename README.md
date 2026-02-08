imp1ort { createPublicClient, http, parseEther } from 'viem';
import { base } from 'viem/chains';
import { createSmartAccountClient } from 'permissionless';
import { privateKeyToSimpleSmartAccount } from 'permissionless/accounts';
import { createPimlicoPaymasterClient } from 'permissionless/clients/pimlico';

// 1. Setup Clients
const publicClient = createPublicClient({
  chain: base,
  transport: http("https://mainnet.base.org")
});

// Replace with your Paymaster RPC from Coinbase Developer Platform or Pimlico
const paymasterContext = {
  url: "https://api.developer.coinbase.com/rpc/v1/base/YOUR_API_KEY" 
};

async function runPayment() {
  // 2. Create the Smart Account (The "Future" Wallet)
  // This uses a private key to OWN the wallet, but the wallet is a contract.
  const simpleAccount = await privateKeyToSimpleSmartAccount(publicClient, {
    privateKey: "0xYOUR_PRIVATE_KEY", // The owner
    factoryAddress: "0x9406Cc6185a346906296840746125a0E44976454", // Base Factory
    entryPoint: "0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789",
  });

  const smartAccountClient = createSmartAccountClient({
    account: simpleAccount,
    chain: base,
    transport: http(paymasterContext.url), // Route through Paymaster
    sponsorUserOperation: async (args) => {
        // This is where the magic happens: The Paymaster signs to pay the gas
        console.log("Sponsoring transaction...");
        return { ...args, paymasterAndData: "0x..." }; 
    }
  });

  // 3. Execute a Gasless Payment
  const txHash = await smartAccountClient.sendTransaction({
    to: "0xRecipientAddress",
    value: parseEther("0.01"), 
  });

  console.log(`Payment sent! View on Basescan: https://basescan.org/tx/${txHash}`);
}

runPayment();
