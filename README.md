
# Supply Chain Fashion Industry


This is a project made with Hyperledger Firefly.  We have four stakeholders, which are: 

![App Screenshot](https://github.com/JMarques98/fashionsupplychain/blob/c60f70d89032de457aa95780095c4e4757426943/resources/FashionSupply.png)

- Factory A
- Factory B
- GOTS (Global Organic Textil Standard)
- Cotton Supplier

## Context

Brands want to have traceability of their products to prove that they collaborate with sustainable fashion with organic products.

So we will have a sample process where the cotton supplier sends the organic textile standards body (GOTS) and the factories information about their cotton harvest to provide more transparency. GOTS reviews the information and rates suppliers over time.

Once the authenticity of the data provided by the supplier has been verified, GOTS sends a message with a certificate to the cotton supplier. 

It is then that the factory proceeds to create its garment from that material and once the garment is created, GOTS issues a tokenized certificate making it clear that the global organic textile standards for garment production have been met. 

Finally, the factory puts the garment on sale and any user can go and buy it, having full traceability of the garment. 

## Requirements

You must have installed on your machine: 

`Docker`

`Docker-engine`

`Go`

## 1. Install Hyperledger Cli

1. Open Terminal
2. Run ‘go version’ to check whether you have installed go
3. If not please install the latest version which is 1.17 now
4. Run ‘go version’again after install to double check installation success
5. Install Docker: https://docs.docker.com/get-docker/
6. Now you are ready to install in a easy way the CLI. Run the following command: 

```bash
go install github.com/hyperledger/firefly-cli/ff@latest

```

If you are in Linux/Mac, you should export GOPATH to be able to use the ff CLI. 


```bash
export GOPATH="$HOME/go" 
PATH="$GOPATH/bin:$PATH"
```

Then, check that you have ff CLI, just run: 


```bash
ff
```

## 2. Set up the Environment

We are now going to create and configure the project with the four participants mentioned above. To do this run the following command: 


```bash
ff init --prompt-names
```

 Now you will see that you will be asked questions for the configuration of the project, such as: 

 - stack name: The name you are going to give to your project
 - number of members: Number of members our network will have
 - name for org 0: Name for the first organization
 - name for node 0: Name for the node of the first organization
 - name for org n .... 

 In our example we will have the following configuration: 


```bash
stack name: fashion
number of members: 4
name for org 0: gots
name for node 0: gots
name for org 1: cotton_supplier
name for node 1: cotton_supplier
name for org 2: fabric_a
name for node 2: fabric_a
name for org 3: fabric_b
name for node 3: fabric_b

```

## 3. We deploy our network
Now that we have everything configured, all we have to do is start our project with a simple command and all the necessary containers for our blockchain application will be deployed. For each organization we will have a Sandbox url from which we can execute transactions, create tokens, execute our own Smart Contracts, etc... And we will also have an explorer for each organization. 

```bash
ff start nameofyourapplication

```

In our case: 

```bash
ff start fashion

```
🥶🥶🥶 Congrats!! You now have your network "ready" to use. 🥶🥶🥶


## ✋🏽🛑 Well... Wait! let's deploy our Supply Chain Smart Contract.

We can find our Smart Contracts inside the folder "Resources/Supply_Fashion.sol".

```bash
// SPDX-License-Identifier: GPL-3.0

pragma solidity >=0.7.0 <0.9.0;


contract SupplyChain {
  // Product Information
  struct Product {
    uint256 productID;
    string productName;
    string productDescription;
    uint256 productPrice;
    address owner;
    bool isAvailable;
  }

  mapping (uint256 => Product) public products;
  uint256 public productCounter;

  // Events
  event ProductCreated(uint256 productID, string productName, string productDescription, uint256 productPrice, address owner);
  event ProductSold(uint256 productID, address buyer);
  event ProductAvailabilityChanged(uint256 productID, bool isAvailable);

  // Functions
  function createProduct(string memory _productName, string memory _productDescription, uint256 _productPrice) public {
    productCounter++;
    products[productCounter] = Product(productCounter, _productName, _productDescription, _productPrice, msg.sender, true);
    emit ProductCreated(productCounter, _productName, _productDescription, _productPrice, msg.sender);
  }

  function sellProduct(uint256 _productID, address _buyer) public {
    Product storage product = products[_productID];
    require(product.isAvailable, "Product is not available for sale.");
    require(product.owner == msg.sender, "You are not the owner of this product.");
    product.isAvailable = false;
    product.owner = _buyer;
    emit ProductSold(_productID, _buyer);
    emit ProductAvailabilityChanged(_productID, false);
  }

  function changeProductAvailability(uint256 _productID, bool _isAvailable) public {
    Product storage product = products[_productID];
    require(product.owner == msg.sender, "You are not the owner of this product.");
    product.isAvailable = _isAvailable;
    emit ProductAvailabilityChanged(_productID, _isAvailable);
  }
}
```
## Deploy the contract
**Now we have to open the Sandbox from Company A (for example):** 

- Go to --> http://127.0.0.1:5309
- Click in "Contracts"
- Go to the terminal and go to the directory inside the folder "resources" and type:  

```bash
solc --combined-json abi,bin Supply_Fashion.sol > Supply_Fashion.json
```

```bash
ff deploy ethereum fashion Supply_Fashion.json
```
And we will have an output in our terminal with the Contract Address: 

```bash
ff deploy ethereum fashion Supply_Fashion.json
```

## Define the contract Interface
Now you have to open the the Supply_Fashion.json in your favourite IDE and copy the ABI, which start from "abi:". It should look like this (you can copy this): 

```bash
[
                {
                    "anonymous": false,
                    "inputs": [
                        {
                            "indexed": false,
                            "internalType": "uint256",
                            "name": "productID",
                            "type": "uint256"
                        },
                        {
                            "indexed": false,
                            "internalType": "bool",
                            "name": "isAvailable",
                            "type": "bool"
                        }
                    ],
                    "name": "ProductAvailabilityChanged",
                    "type": "event"
                },
                {
                    "anonymous": false,
                    "inputs": [
                        {
                            "indexed": false,
                            "internalType": "uint256",
                            "name": "productID",
                            "type": "uint256"
                        },
                        {
                            "indexed": false,
                            "internalType": "string",
                            "name": "productName",
                            "type": "string"
                        },
                        {
                            "indexed": false,
                            "internalType": "string",
                            "name": "productDescription",
                            "type": "string"
                        },
                        {
                            "indexed": false,
                            "internalType": "uint256",
                            "name": "productPrice",
                            "type": "uint256"
                        },
                        {
                            "indexed": false,
                            "internalType": "address",
                            "name": "owner",
                            "type": "address"
                        }
                    ],
                    "name": "ProductCreated",
                    "type": "event"
                },
                {
                    "anonymous": false,
                    "inputs": [
                        {
                            "indexed": false,
                            "internalType": "uint256",
                            "name": "productID",
                            "type": "uint256"
                        },
                        {
                            "indexed": false,
                            "internalType": "address",
                            "name": "buyer",
                            "type": "address"
                        }
                    ],
                    "name": "ProductSold",
                    "type": "event"
                },
                {
                    "inputs": [
                        {
                            "internalType": "uint256",
                            "name": "_productID",
                            "type": "uint256"
                        },
                        {
                            "internalType": "bool",
                            "name": "_isAvailable",
                            "type": "bool"
                        }
                    ],
                    "name": "changeProductAvailability",
                    "outputs": [],
                    "stateMutability": "nonpayable",
                    "type": "function"
                },
                {
                    "inputs": [
                        {
                            "internalType": "string",
                            "name": "_productName",
                            "type": "string"
                        },
                        {
                            "internalType": "string",
                            "name": "_productDescription",
                            "type": "string"
                        },
                        {
                            "internalType": "uint256",
                            "name": "_productPrice",
                            "type": "uint256"
                        }
                    ],
                    "name": "createProduct",
                    "outputs": [],
                    "stateMutability": "nonpayable",
                    "type": "function"
                },
                {
                    "inputs": [],
                    "name": "productCounter",
                    "outputs": [
                        {
                            "internalType": "uint256",
                            "name": "",
                            "type": "uint256"
                        }
                    ],
                    "stateMutability": "view",
                    "type": "function"
                },
                {
                    "inputs": [
                        {
                            "internalType": "uint256",
                            "name": "",
                            "type": "uint256"
                        }
                    ],
                    "name": "products",
                    "outputs": [
                        {
                            "internalType": "uint256",
                            "name": "productID",
                            "type": "uint256"
                        },
                        {
                            "internalType": "string",
                            "name": "productName",
                            "type": "string"
                        },
                        {
                            "internalType": "string",
                            "name": "productDescription",
                            "type": "string"
                        },
                        {
                            "internalType": "uint256",
                            "name": "productPrice",
                            "type": "uint256"
                        },
                        {
                            "internalType": "address",
                            "name": "owner",
                            "type": "address"
                        },
                        {
                            "internalType": "bool",
                            "name": "isAvailable",
                            "type": "bool"
                        }
                    ],
                    "stateMutability": "view",
                    "type": "function"
                },
                {
                    "inputs": [
                        {
                            "internalType": "uint256",
                            "name": "_productID",
                            "type": "uint256"
                        },
                        {
                            "internalType": "address",
                            "name": "_buyer",
                            "type": "address"
                        }
                    ],
                    "name": "sellProduct",
                    "outputs": [],
                    "stateMutability": "nonpayable",
                    "type": "function"
                }
            ]
```

Once you have copied this, you have to go to the Sandbox and paste "Schema" field:

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

## Register the Contract API 

Finally, copy the Smart Contract address displayed on the terminal before and give the name for Swagger API. 

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

# Register the different Events

![App Screenshot](https://via.placeholder.com/468x300?text=App+Screenshot+Here)

- Tags - To be able to differentiate the different events within the network and that the different organizations can act according to the requirements they want. This means that if for example we put a New_Research tag and an organization has configured that when an event arrives with that tag it will do a certain action, such as sending the reseach to the research team, it will do it, but if it arrives with the Last_News tag and it is not interested, it will not listen to the event. 

- Topic - is so that from firefly the events will be ejected in an orderly way in a correct way.
