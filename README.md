# Demo_TraSSI_ICSOC

To simplify the task for researchers in reproducing our TraSSI solution, we have chosen to rely on online tools that avoid the need for complex local installations.

- For ZoKrates, instead of requiring users to manually install and configure the system, we utilize the ZoKrates Plugin within the Ethereum Remix online IDE, offering a more convenient and seamless way to interact with the tool.
  
  https://remix.ethereum.org/
  
- For FHE computations, the Python scripts are available and ready to run directly through a Colab link, provided in a ".pynb" notebook format. This approach ensures easy execution and accessibility, significantly reducing installation barriers and streamlining the reproduction process for researchers.
  
  https://colab.research.google.com/drive/1HK18ven1k0utCf6WnSoBIxQOCo6usO4d


## Step1: ZKP-based Verifiable Computations (Part1)

We provide a step-by-step guide to assist you in compiling, setting up, exporting the verifier, computing the witness, and generating proofs.

For documentation on ZoKrates open the [link](https://zokrates.github.io/gettingstarted.html).

### 1. Install the Zokrates Plugin

Open the [Remix IDE](https://remix.ethereum.org/) in a browser window.

Navigate to the Plugin Manager and activate the following plugins:

- ZoKrates

- Solidity Compiler (already activated)

- Deploy & Run Transactions (already activated)

Once the plugins are activated, go to the ZoKrates plugin and click on the example hyperlink, accepting any necessary permissions.

This will prepare your environment for the subsequent tasks.

![2](images/2.png)

![3](images/3.png) 

![7](images/7.png)


### 2. Compile

To proceed, copy the entire program code from the designated source.

``` python
from "ecc/babyjubjubParams" import BabyJubJubParams;
import "ecc/babyjubjubParams" as context;
import "ecc/proofOfOwnership" as proofOfOwnership;
import "hashes/sha256/512bitPacked" as sha256packed;

def proofOfKnowledge(field[4] secret, field[2] hash) -> bool {
  return (hash == sha256packed(secret));
  }

def verifyThreashold(field revenue, field max_Threashold) -> bool {
    // Check if the revenue exceeds the max_threashold
    return (revenue <= max_Threashold);
}
  
def main(field[2] pkA, field[2] pkB, field[2] hash, private field skA, private field[4] secret, private field revenue, field max_Threashold, private field skB) -> bool {
  BabyJubJubParams context = context();
  bool orgAhasKnowledge = proofOfKnowledge(secret, hash);
  bool orgAhasOwnership = proofOfOwnership(pkA, skA, context);
  bool audhasOwnership = proofOfOwnership(pkB, skB, context);
  bool isOrgAwithKnowledge = (orgAhasKnowledge && orgAhasOwnership);
  bool out1 = (isOrgAwithKnowledge == true || audhasOwnership == true);
  bool out2 = verifyThreashold(revenue , max_Threashold);
  return (out1 && out2);
  }
```

To compile our program, click on the "Compile" button.

If the process completes without errors, a message indicating successful compilation will be displayed.

![11](images/11.png)


### 3. Setup

The next step is to perform the setup. Click on "Setup" to expand the section, then select "Run Setup."

If the setup is successful, a confirmation message will appear. You can download the proving and verifying keys by clicking the "Download Keys" button.

![13](images/13.png)


### 4. Export Verifier

We move to the Export Verifier step.

If the export is successful, you will have the option to either open the generated verifier directly in the Remix Editor or download it. By selecting the "Open in Remix Editor" option, the plugin will automatically create and open the "verifier.sol" file within the Remix Editor. 

Instructions for compiling and deploying the Verifier smart contract are provided further on.

![16](images/16.png)



### 5.Compile and Deploy "Verifier.sol"

To compile and deploy the "Verifier.sol" contract, begin by navigating to the Solidity Compiler plugin and selecting "Compile" "Verifier.sol."

Once the compilation is complete, switch to the "Deploy & Run Transactions" plugin. From the dropdown menu, select "Verifier — browser/verifier.sol" and click "Deploy." You can choose to deploy the contract on the JavaScript VM (Remix VM) or any testnet or mainnet. Please note that these contracts are large and may consume significant gas fees.

If you select the JavaScript VM (Remix VM), you do not need to use MetaMask. However, if you choose to deploy on a testnet like Sepolia, you will need to use MetaMask, as illustrated in the figure below.
(You can simply use JavaScript VM (Remix VM)).


![30](images/30.png)
![37](images/37.png)
![38](images/38.png)
![40](images/40.png)
![42](images/42.png)
![43](images/43.png)
![44](images/44.png)

After deploying the verifier, we will proceed to the "Compute Witness" and "Generate Proof" steps in ZoKrates. However, before we can compute the witness, we need to obtain the necessary "inputs" from the FHE calculations. Therefore, the next step is to run the Python code for the FHE computations first.




## Step2: FHE-based Computations on Encrypted Data (Part1)

The FHE computations are provided in this Colab notebook, accessible via the following [link](https://colab.research.google.com/drive/1HK18ven1k0utCf6WnSoBIxQOCo6usO4d). This allows anyone to directly test and execute the code without the need for local installations. The notebook is fully set up, enabling researchers to run the computations with ease and explore the functionality of the homomorphic encryption techniques used in our solution.

Link: https://colab.research.google.com/drive/1HK18ven1k0utCf6WnSoBIxQOCo6usO4d

### Install the required Python libraries by running the following commands:

These libraries are essential for enabling ZoKrates cryptographic functions (zokrates_pycrypto), performing homomorphic encryption operations (tenseal), and interacting with the Ethereum blockchain (web3).

``` bash
pip install zokrates_pycrypto
pip install tenseal
pip install web3
```

### Execute the following code:

``` python
import tenseal as ts
import hashlib
from zokrates_pycrypto.eddsa import PrivateKey, PublicKey
from zokrates_pycrypto.field import FQ
from zokrates_pycrypto.utils import write_signature_for_zokrates_cli

if __name__ == "__main__":

    # Setup TenSEAL context
    context = ts.context(
            ts.SCHEME_TYPE.CKKS,
            poly_modulus_degree=8192,
            coeff_mod_bit_sizes=[60, 40, 40, 60]
          )
    context.generate_galois_keys()
    context.global_scale = 2**40

    # Encrypt the number of units sold
    num_units = [100]
    encrypted_num_units = ts.ckks_vector(context, num_units)

    # Encrypt the price per unit
    price_per_unit = [50]
    encrypted_price_per_unit = ts.ckks_vector(context,price_per_unit)

    # Compute the total revenue
    encrypted_revenue = encrypted_num_units * encrypted_price_per_unit

    # Setup maximum threshold
    max_threshold = "7000"



    # Write witness arguments to disk / Serialize the decrypted result and max_threshold for ZoKrates input
    path = "zokrates_inputs.txt"
    #witness = "" + encrypted_revenue.decrypt()[0] + " " + max_threshold
    witness = str(round(encrypted_revenue.decrypt()[0])) + " " + max_threshold
    with open(path, "w+") as f:
        f.write("".join(witness))

    print(str(round(encrypted_revenue.decrypt()[0])) + " " + max_threshold)

```
