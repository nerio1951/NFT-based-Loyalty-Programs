# NFT-based-Loyalty-Programs
利用NFT创建创新的忠诚度计划，为客户提供独特的奖励和体验。
from web3 import Web3
from solcx import compile_source
from web3.contract import ConciseContract

# Solidity source code for a simple NFT
solidity_src = """
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract LoyaltyNFT is ERC721, Ownable {
    constructor() ERC721("LoyaltyNFT", "LFT") {}

    function mint(address to, uint256 tokenId) public onlyOwner {
        _mint(to, tokenId);
    }
}
"""

# Compile the contract
compiled_sol = compile_source(solidity_src, output_values=["abi", "bin"])
contract_id, contract_interface = compiled_sol.popitem()

# web3.py instance
w3 = Web3(Web3.HTTPProvider('<Your_Ethereum_Node_URL>'))

# Set pre-funded account as sender
w3.eth.defaultAccount = w3.eth.account.privateKeyToAccount('<Your_Private_Key>').address

# Deploy the contract
LoyaltyNFT = w3.eth.contract(abi=contract_interface['abi'], bytecode=contract_interface['bin'])
tx_hash = LoyaltyNFT.constructor().transact()
tx_receipt = w3.eth.waitForTransactionReceipt(tx_hash)

# Contract instance in order to interact with the smart contract
loyalty_nft = w3.eth.contract(
    address=tx_receipt.contractAddress,
    abi=contract_interface['abi'],
)

# Function to mint an NFT to a customer
def mint_nft_to_customer(customer_address, token_id):
    tx_hash = loyalty_nft.functions.mint(customer_address, token_id).transact()
    w3.eth.waitForTransactionReceipt(tx_hash)
    print(f"NFT with token ID {token_id} has been minted to {customer_address}")

# Example: Minting an NFT to a customer
customer_wallet_address = "<Customer_Wallet_Address>"
mint_nft_to_customer(customer_wallet_address, 1)

# Output success message
print("Loyalty NFT minted successfully!")
