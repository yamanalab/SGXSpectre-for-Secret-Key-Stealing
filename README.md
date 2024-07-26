# SGXSpectre for Secret Key Stealing
This repository demonstrates a proof-of-concept attack leveraging a Spectre-like vulnerability to steal a secret key of homomorphic encryption (HE) from an Intel SGX enclave. This work extends the `SGXSpectre` attack originally developed by Dan O'Keeffe, Divya Muthukumaran, Pierre-Louis Aublin, Florian Kelbert, Christian Priebe, Josh Lind, Huanzhou Zhu, and Peter Pietzuch.

## Overview
In this project, we explore the security implications of speculative execution side channels for Intel SGX enclaves, specifically focusing on the HE-secret keys stealing. Our proof-of-concept attack shows that it is possible to extract sensitive data, such as secret keys, from within an SGX enclave using speculative execution side channels.

## Attack Outline
The attack builds on the concepts of the conditional branch misprediction [Spectre attack](https://spectreattack.com/spectre.pdf) by Kocher et al. Our implementation relocates the secret key and the victim function inside the enclave, while the attacker code remains outside the enclave. The attacker code executes the victim function via ecalls, using speculative execution to leak the secret key.

## Code Layout
* `main/main.cpp`: Contains the untrusted code responsible for creating the enclave and mounting the SGXSpectre attack.
* `enclave/enclave_attack.c`: Contains the enclave's secret key and the victim function.

## Caveats
* The `array1_size` variable used to verify the bounds of the index `x` must not be cached. In this proof-of-concept, `array1_size` is stored outside the enclave, allowing the attacker to flush it before each invocation of the victim function. For a secure implementation, this value should be stored within the enclave.
* The `array2` array, probed by the attacker, is kept outside the enclave for simplicity. As noted in the [Spectre paper](https://spectreattack.com/spectre.pdf), a prime+probe attack could be used to infer accesses to `array2` if it were inside the enclave.

# Tested Environment
* CPU: Intel Core i7-8700
* g++==9.4.0
* SGXdriver==2.11
* SGXSDK==2.24.100.3
* SEAL==4.1.1
* OS: Ubuntu 20.04
* kernel: 5.15.0-113-generic

## How to Run the Code
1. Install the Intel(R) SGX SDK for Linux* OS.
2. Install [Microsoft SEAL](https://github.com/Microsoft/SEAL.git) (version 4.1.1)
3. Get the original SGXSpectre code
   ```terminal
   git submodule init
   git submodule update
   ```
4. Copy the original SGXSpectre code and apply our patch
   ```terminal
   cp -r spectre-attack-sgx/SGXSpectre src
   patch -p1 -d src < change.diff
   ```
5. Make the program
   ```terminal
   cd src
   make PREFIX_FOR_SEAL=${HOME}/.local
   ```
   * `PREFIX_FOR_SEAL` is set to the path to the directory where Microsoft SEAL is installed (the path is the same as the value of `-DMAKE_INSTALL_PREFIX` option for building Microsoft SEAL.)
   * Use `SGX_MODE=SIM` for `make` to execute the program with simulation mode.
   * Clean the build before switching modes by `make clean`
6. Configure key size:
   * Modify the values of `N` and `LEN` in `utils.hpp` in `main` directory and `enclave_attack.c` in `enclave` directory.
   * Copy `parms`, `secret_key`, and `ciphertext` in the corresponding directory, such as `64KiB`, to the current directory.
7. Execute the binary directly
   ```terminal
   ./sgxspectre
   ```

## Acknowledgements
This project is based on the original `SGXSpectre` attack by Dan O'Keeffe, Divya Muthukumaran, Pierre-Louis Aublin, Florian Kelbert, Christian Priebe, Josh Lind, Huanzhou Zhu, and Peter Pietzuch from the LSDS group at Imperial College London.

## References
- [SERECA Project](https://lsds.doc.ic.ac.uk/projects/sereca)
- [Spectre Attack Paper](https://spectreattack.com/spectre.pdf)

