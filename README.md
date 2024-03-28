# Decentralized-Crowdfunding
Build a WEB3-based crowdfunding platform that uses smart contracts to ensure transparency and automate the distribution of funds.
from web3 import Web3
from solcx import compile_standard, install_solc
import json

# Install the Solidity compiler
install_solc('0.8.0')

# Compile the contract
with open('Crowdfunding.sol', 'r') as file:
    crowdfunding_source_code = file.read()

compiled_sol = compile_standard({
    "language": "Solidity",
    "sources": {"Crowdfunding.sol": {"content": crowdfunding_source_code}},
    "settings": {
        "outputSelection": {
            "*": {
                "*": ["abi", "metadata", "evm.bytecode", "evm.bytecode.sourceMap"]
            }
        }
    }
},
solc_version='0.8.0',
)

# Extract bytecode and ABI
bytecode = compiled_sol['contracts']['Crowdfunding.sol']['Crowdfunding']['evm']['bytecode']['object']
abi = json.loads(compiled_sol['contracts']['Crowdfunding.sol']['Crowdfunding']['metadata'])['output']['abi']

# Connect to local Ethereum node
w3 = Web3(Web3.HTTPProvider('http://127.0.0.1:7545'))
w3.eth.default_account = w3.eth.accounts[0]

# Deploy the contract
Crowdfunding = w3.eth.contract(abi=abi, bytecode=bytecode)
tx_hash = Crowdfunding.constructor().transact()
tx_receipt = w3.eth.wait_for_transaction_receipt(tx_hash)
crowdfunding = w3.eth.contract(address=tx_receipt.contractAddress, abi=abi)

print(f'Contract deployed at {tx_receipt.contractAddress}')

# Example usage
def create_campaign(title, description, goal, duration):
    tx_hash = crowdfunding.functions.createCampaign(title, description, goal, duration).transact()
    w3.eth.wait
