import hashlib
import json
import time

class Blockchain:
    def __init__(self):
        self.chain = []
        self.transactions = []
        self.create_block(previous_hash='0')

    def create_block(self, previous_hash):
        block = {
            'index': len(self.chain) + 1,
            'timestamp': time.time(),
            'transactions': self.transactions,
            'previous_hash': previous_hash,
            'nonce': 0
        }
        block['hash'] = self.hash_block(block)
        self.transactions = []
        self.chain.append(block)
        return block

    def hash_block(self, block):
        block_string = json.dumps(block, sort_keys=True).encode()
        return hashlib.sha256(block_string).hexdigest()

    def get_last_block(self):
        return self.chain[-1]

    def proof_of_work(self, block, difficulty=4):
        block['nonce'] = 0
        calculated_hash = self.hash_block(block)
        while not calculated_hash.startswith('0' * difficulty):
            block['nonce'] += 1
            calculated_hash = self.hash_block(block)
        return calculated_hash

    def add_transaction(self, sender, recipient, amount):
        self.transactions.append({
            'sender': sender,
            'recipient': recipient,
            'amount': amount
        })
        return self.get_last_block()['index'] + 1

    def is_chain_valid(self):
        for i in range(1, len(self.chain)):
            current_block = self.chain[i]
            previous_block = self.chain[i - 1]

            # Check if current block's hash is valid
            if current_block['previous_hash'] != previous_block['hash']:
                return False

            # Check if current block's hash starts with enough leading zeros (proof of work)
            if not current_block['hash'].startswith('0000'):
                return False

        return True

if __name__ == "__main__":
    blockchain = Blockchain()
    
    # Add some transactions
    blockchain.add_transaction(sender="Alice", recipient="Bob", amount=50)
    blockchain.add_transaction(sender="Bob", recipient="Charlie", amount=25)
    
    # Mine a new block
    last_block = blockchain.get_last_block()
    new_hash = blockchain.proof_of_work(last_block)
    blockchain.create_block(previous_hash=new_hash)

    print("Blockchain valid:", blockchain.is_chain_valid())
    print(json.dumps(blockchain.chain, indent=4))
