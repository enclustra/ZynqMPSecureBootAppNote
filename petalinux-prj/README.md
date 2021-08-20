# PetaLinux Project

This description shows how to create an encrypted and authenticated boot image for the Enclustra Mercury XU5 module.

## Folder and files description

| Folder/File name    | Description                                                                                                |
| ------------------- | ---------------------------------------------------------------------------------------------------------- |
| uboot-scripts       | Contains u-boot scripts to write images into the QSPI memory section 0x0-0x1000000 or 0x1000000-0x2000000. |
| boot.bif            | Boot image file. Contains information how the final image is packed.                                       |
| make_uboot_scrit.sh | Shell script to build the u-boot scripts.                                                                  |
| moverootfs.sh       | Shell script to move generated rootfs to /tftpboot/nfsroot if nfs is used.                                 |

## Quickstart Guide

This guide describes how to set up this project as quickly as possible. First
install PetaLinux 2020.1 and Bootgen 2020.1 on your host. Information on how to install PetaLinux can be
found in the documentation linked [here](#getting-started).


1. Get the .BSP file (Board support package) for your module and baseboard from [https://github.com/enclustra](https://github.com/enclustra). For example, the .BSP for the XU5 module and the PE1 baseboard can be found [here](https://github.com/enclustra/Mercury_XU5_PE1_Reference_Design/releases/tag/2020.1_v1.1.0).

2. Source the Petalinux environment script:

    ```sh
    source /<path-to-petalinux-installation-dir>/settings.sh
    ```

3. Create the Petalinux project:

    ```sh
    petalinux-create --type project -s <path-to-bsp-file>.bsp -n petalinux-project
    ```
	
4. Move to the `petalinux-project` folder.

    ```sh
    cd petalinux-project
    ```

5. Generate the executables. The executables will be generated in `images/linux`.

    ```sh
    petalinux-build
    ```

7. Generate keys for your image, as no keys are included in the project. With
   the standard `.bif` file, four AES and 5 RSA keys have to be generated.
   First we are going to generate the 5 RSA keys with the following command.

    ```sh
    # Create a directory for your keys called "keys"
    mkdir keys
    # Change into the keys directory
    cd keys

# Generate 5 keys with a length of 4096
    openssl genrsa -out primary.pem 4096 
    openssl genrsa -out secondary0.pem 4096 
    openssl genrsa -out secondary1.pem 4096 
    openssl genrsa -out secondary2.pem 4096 
    openssl genrsa -out secondary3.pem 4096 
    ```

8. Next we are going to generate the AES key files.

    ```sh
    # Change back to the project directory
    cd ..
    # Generate all keys by running bootgen through the petalinux-package command
    petalinux-package --boot --bif boot.bif --bootgen-extra-args "-p\ mercuryxu5" --force
    ```

9. Generate the PPK hash, which will later be stored in the eFuse.

    ```sh
    petalinux-package --boot --bif boot.bif --bootgen-extra-args "-efuseppkbits\ keys/efuseppkhash.txt" --force
    ```

9. Package the boot image

    ```sh
    petalinux-package --boot --fsbl images/linux/zynqmp_fsbl.elf --u-boot images/linux/u-boot.elf --pmufw images/linux/pmufw.elf --fpga images/linux/system.bit --force

    ```

10. Write at least the AES key to the BBRAM following this
    [README](../program-efuse-bbram/README.md) in the folder
    `program-efuse-bbram`. If the PPK hash is not stored inside the eFuse,
    `bh_auth_enable` has to be added to the `.bif` file. It enables simulation
    of the authentication process for development.
    
    ```c
    [fsbl_config] opt_key, bh_auth_enable
    ```

11. Copy the files `BOOT.bin, boot.scr, Image and system.dtb` found in `images/linux` to the first partition of the sd card and extract the file `rootfs.tar.gz` to the second ext3 partition. This [guide](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_1/ug1144-petalinux-tools-reference-guide.pdf) describes these steps in more detail
 
12. Insert the sd card and boot the image. 
 

 
## PetaLinux project structure

A useful PetaLinux project structure description can be found in the Tools
Documentation and Reference Guise.

[Petalinux Tools Documentation Reference Guide (UG1144)](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_1/ug1144-petalinux-tools-reference-guide.pdf)

## PetaLinux or Yocto?

PetaLinux is the tool chain from Xilinx to create a Linux image for the Zynq
Ultrascale+. It is built upon Yocto, which is an-open source project, to
create Linux distributions for different platforms. Instead of using PetaLinux,
it is possible to use Yocto  for development on the Zynq Ultrascale+.

The advantage of Yocto is, it works for many platforms and therefore, the same
tools and workflow can be used for different platforms. Because Yocto is more
often used, it has a better documentation, more examples and more guides
available than for PetaLinux. 

However, there is also a disadvantage. Although Xilinx provides guides to work
with Yocto, it is not officially supported by Xilinx. If any problem occurs,
even if it is related to Yocto, Xilinx will not provide any support for it.
Xilinx maintains a forum for PetaLinux, where most questions are answered.
Further, guides and examples for Yocto are not as good maintained as for
PetaLinux and are sometimes out of date.

## Getting started

This section describes how to create the first project for a Mercury Board from
Enclustra. A description of how to install and use PetaLinux can be found in the
following user manuals and application notes from Xilinx. The following Guides
focuses on the specialties of the Mercury Board.

This guide was tested with Petalinux 2020.1 and the Enclustra Petalinux BSPs for 2020.1.

  - [Petalinux Tools Documentation Reference Guide (UG1144)](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2020_1/ug1144-petalinux-tools-reference-guide.pdf)
  - [PetaLinux Tools Documentation Command Line Reference Guide (UG1157)](https://www.xilinx.com/support/documentation/sw_manuals/xilinx2019_1/ug1157-petalinux-tools-command-line-guide.pdf)
  - [Xilinx Wiki](https://xilinx-wiki.atlassian.net/wiki/spaces/A/overview)

### Create Vivado project

If further adjustments to the hardware have to be made, it makes sense to create
a Vivado project as well. The user guide to the reference design from Enclustra 
describes how to create a Vivado project.

  - Mercury XU5 SoC Module Reference Design for Mercury+ PE1 Base Board User
    Manual 

### Export Hardware

After synthesizing the Vivado project, the hardware information and the
bitstream have to be exported. To export the hardware access

```
File->Export->Export Hardware...
```

in the menu bar. It is essential to check `Include bitstream` in the options
window, in order to use the export in PetaLinux. An .XSA file is created, which at a later step needs to be imported in the Petalinux project.

### Create Petalinux Project

A BSP (board support package) contains the required configuration and settings for a specific module and baseboard. The BSPs for Enclustra modules can be downloaded [here](https://github.com/enclustra). 

    ```sh
    petalinux-create --type project -s <path-to-bsp-file>.bsp -n petalinux-project
    ```

### Import hardware to the PetaLinux project

This step is always necessary, after any changes in hardware. The new bitstream has to be exported and imported to PetaLinux. The bitstream which is included in the BSP is generated with the Enclustra reference design. In case any change is made to the reference design or a different design is used, the .XSA file needs to be imported again. 

It is essential only to have one exported hardware file in the folder, else
PetaLinux will complain. Import the hardware description file to PetaLinux with
the following command.

```sh
petalinux-config --get-hw-description=<PATH-TO-FOLDER-CONTAINING-XSA>
```

### Configure PetaLinux Project

After importing the hardware description, the first configuring window appears.
It can be exited by double-pressing the `ESC` key. The different components are
configured separately in PetaLinux. There are two ways to configure components.
If available, a `menuconfig` can be invoked, as seen after the hardware import, by typing:

```sh
petalinux-config -c <COMPONENT-NAME>
```

The different components with are:

  - `u-boot`
  - `kernel`
  - `rootfs` (default)

Other components can be configured by creating a configuration file.
These components are:

  - `pmufw`
  - `bootloader` — FSBL 
  - `device-tree`

### Build PetaLinux project

Likewise, to configuring, build individual components with the `-c <COMPONENT-NAME>` flag. To build
the whole project use:

```sh
petalinux-build
```

### Packaging the project

After building the project, all individual executables are generated
individually but are not bundled together in a boot-able image. To generate a
basic boot-able image use:

```sh
petalinux-package --boot --fsbl <FSBL-ELF> --fpga <BITSTREAM> --u-boot --pmufw <PMUFW-ELF>
```

With this image, the device will not boot securely. To generate an image for a
secure boot, read the following section.

# Secure Boot

The implementation of secure boot focuses mainly on packaging and settings in
the eFuse registers. Two implementations of secure boot are possible. By setting
the `ENC_ONLY` bit in the eFuse the “encryption only” secure boot process is activated.
This boot mode is limiting, because this mode only uses AES for authentication
and confidentiality. Additionally, the device can only use the AES master or
device key stored in the eFuse register. Therefore, the other secure boot mode
“root of trust” has been implemented. To activate the “root of trust” mode one
has to write the `RSA_EN` bits in the eFuse. For testing purposes, Xilinx provides a
feature to work with this secure boot mode, before writing the `RSA_EN` bits. In this
mode, everything works as expected, with authentication, but the authentication
of the PPK never happens. Instead, the bootloader skips this step and goes right
to the authentication process. Therefore, neither secure boot nor key revocation
works, because the checks between eFuse and the boot header are skipped.
Nevertheless, the chip is not limited to authenticated images. It can run any
images, as long as those secure boot bits are not set.

Too boot securely, the content of the boot-able image and the security features
of each partition have to be defined. Instead of packaging the image only using
the command above, which is very limited, an additional file is introduced. This
`.bif` file is a description file of the boot image. It defines:

  - The containing partitions
  - Where the partition is executed
  - If the partitions are encrypted, what key to use
  - If the partitions are authenticated, what key to use
  - Where and when partitions are loaded

A complete list and description of all definitions is available in the bootgen
User Guide . bootgen is the tool behind the `.bif` file. It is automatically
installed when installing PetaLinux. In case the bootgen tool has to be used
separately, it can be installed using the Vivado Design Suite Web installer .

## Authentication

The key and signature paths have to be included in order to generate images with
authentication. Thus, in the `.bif` file has to be declared, which key is used for
which partition. Because RSA is an asymmetric authentication method with public
and private keys, there are multiple possibilities to do so. The more
straightforward way, which we implemented, was by providing the secret keys. The
other possibility is by using only public keys. This possibility can be used to
protect the secret keys. The following code listing shows the settings when only
public keys are used.

```c
image : {
    /* Key revocation features */
    [auth_params] ppk_select=0; spk_select=spk-efuse; spk_id=0x00000000

    /* Define primary public key file */
    [ppkfile]primarypublickey.pem

    /* Define secondary public key file for the boot header*/
    [spkfile]secondarypublickey1.pem

    /* Define the signature file for the secondary public key */
    [spksignature]spk_signature.sig

    /* Define the signature file for the boot header and fsbl as they are together authenticated */
    [bhsignature]bh_signature.sig

    /* Define the signature file for the header table */
    [headersignature] header_signature.sig

    /* Enabling authentication and define signature file for individual images */
    [
    authentication    = rsa,
    spkfile           = secondarypublickey2.pem,
    presign           = signature2.sig,
    /* (Optional) Select a different spk_id and spk_id source */
    spk_select        = <spk-efuse/user-efuse>,
    spk_id            = 0x00000000
    ] image.bin
}
```

This way is more complicated than with secret keys because additionally, a
signature has to be provided. However, it is a more secure way because the
secret keys remain unrevealed. To generate the signatures and key files, use a
toolkit like openssl. The generated keys have to have a key length of 4096 bits.
A good systematic description is available in the bootgen documentation  in the
description about creating images using a hsm module, starting on page 73.

If the secret keys are available during the image creation, the `.bif` file looks a bit
simpler. Instead of defining public keys and signatures, only the secret keys
have to be defined. bootgen automatically generates signatures and public keys
in the process.

```c
image : {
    /* Key revocation features */
    [auth_params] ppk_select=0; spk_select=spk-efuse; spk_id=0x00000000

    /* Define primary secret key */
    [pskfile] primarysecretkey.pem

    /* Define secondary secret key for the boot header*/
    [sskfile] secondarysecretkey1.pem

    /* Enabling authentication and define a secondary secret key for individual images */
    [
    authentication    = rsa,
    sskfile           = secondarysecretkey2.pem,
    /* (Optional) Select a different spk_id and spk_id source */
    spk_select        = <spk-efuse/user-efuse>,
    spk_id            = 0x00000000
    ] image.bin
}
```

In this example, for each partition, a different key has been used. The
additional parameter

```c
[fsbl_config] bh_auth_enable
```

can be set, to test authentication and secure boot without blowing the `RSA_EN`
eFuse. With this parameter set, authentication will be enabled. However, the PPK
will not be checked with the hash stored in the eFuse, as well as the `SPK_IDs`.
If neither the `RSA_EN` eFuse nor this parameter is set, nothing of the boot
image will be authenticated.

**Important**
The boot header and the fsbl need to have the same key defined. They usually are
authenticated separately, but we noticed during testing, that the authentication
throws an error, if two separate keys are defined.

### Generate Keys for RSA 

In both cases, at some point, keys have to be generated. Xilinx provides a way
of using bootgen.

```sh
openssl genrsa -out key.pem 4096
```

With this way, bootgen will generate secret RSA keys in the defined location.
The location must exist, else bootgen will throw a segmentation fault. This
command is only usable if a primary and one secondary key is used. In cases,
which more secondary keys are used, bootgen will not generate the other keys.
Thus, the second approach with openssl is easier.

Important is to generate a key with the key length of 4096 bits.

### Generate PPK Hash

To be able to write the hash from the PPK to the eFuse, it first has to be
generated. Thus, the bootgen tool also provides a feature to generate the hash
automatically.

```sh
bootgen -efuseppkbits efuseppkhash.txt -arch zynqmp -w -o test.bin -image boot.bif
```

Bootgen stores the hash in the defined file. In this example `efuseppkhash.txt`.

## Encryption

As AES is a symmetrical method, it is much easier to implement. However, it has
the disadvantage that the key files have to be available, for the generation of
the image. The operational key method reduces the amount, in which the device
key is used. While with the normal method, all the images would be decrypted
using the device key, with the operational key method, only a small encrypted
part of the boot header is decrypted with the device key. All following
partitions are decrypted using their own key, saved in the previous partition.
To further reduce the risk of a key leakage, the rolling key method can also be
implemented. The following code listing shows a basic `.bif` file, implementing
encryption with the operational key method and the rolling key method for the
`longimage.bit` file. More information about file attributes can be found in the
bootgen User Guide.

```c
image : {
    /* Define source of device key */
    [keysrc_encryption] BBRAM_red_key

    /* Further options */
    [fsbl_config] opt_key

    /* Enabling encryption and defining key file for individual images */
    [
    encryption    = AES,
    AESkeyfile    = AESkeyfile1.nky
    ] image.bin

    /* (Optional) Enabling rolling key method for big image partition */
    [
    encryption    = AES,
    /* Attention!: the keyfile has to have the number of keys required for the amount of blocks */
    AESkeyfile    = AESkyfile2.nky,
    /* Define length and amount of blocks */
    blocks        = 2014(2);2048(2);8192(2);4096(*)
    ] longimage.bit
  }
```

### Generate Keys for AES

To generate keys for AES nothing has to be especially done. When the bootgen
command runs, it automatically generates new keys, if it does not find the
defined ones. Only the part name has to be specified with the option, in
addition to the usual options. The part name can be anything. The follwing code
listing shows that bootgen writes the part name into the key file. The key files
may vary. This key file has been generated with the `boot.bif` file from this
repository.

```
Device       <partname>;

  Key 0        50C0F949817F0A00DD3A66117599936D7A14BAB349BB546E1CBBAD32F69278D3;
  IV 0         A2E411F57D6045FE506B994F;

  Key 1        83283F2FCA0F1A2643A8D62811CBB2DB6184C330CF9E926818FA098556ECBBD5;
  IV 1         C28E78DAE84F9BF415FD142D;

  Key Opt      96EEBD73F10EB683E8F4028D609AB86AA90E8D19920AB55BF507878C6DEE3E86;
```


The first key is the device key, which has to be loaded, to the eFuse or BBRAM.
This key should be the same across all generated key files. Also, the according
iv should be the same. The second key is the individual key for each partition.
This key and the iv are different from partition to partition. The last key is
the operational key. This key and iv are also identical on all key files.
