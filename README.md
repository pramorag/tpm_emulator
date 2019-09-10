# tpm_emulator

Steps on downloading, building and running TPM 2.0 emulator

This section details the steps involved in installing the TPM 2.0 emulator on Ubuntu 18.10 virtual machine running inside of VMWare Fusion or VirtualBox.

- Linux kernel version in Ubuntu 18.10 is 4.18.0
- Hard disk partition size = 200GB and 2GB of RAM for the virtual machine

TPM 2.0 emulator sources
-------------------------
1) https://github.com/stefanberger/swtpm.git
2) https://sourceforge.net/projects/ibmswtpm2/files/ibmtpm1332.tar.gz/download

For the steps below, we will use SWTPM source from Stefan (from (1) above)

Various open source software needed to get a working environment for TPM2.0 emulation
--------------------------------------------------------------------------------------

1) libtpms (used by TPM 2.0 emulator)
2) swtpm (TPM 2.0 emulator)
3) tpm2-tss (TCG TPM2 software stack)
4) tpm2-tools (some TPM2 tools to talk to TPM 2.0 emulator using tpm2-tss)

A good book to understand TPM
- https://link.springer.com/content/pdf/10.1007%2F978-1-4302-6584-9.pdf

Order of software installation and dependent packages needed
------------------------------------------------------------

Various libraries / packages needed to install TPM 2.0 emulator, its dependent libraries and TSS tools:

- sudo apt install automake
- sudo apt install autoconf
- sudo apt install bash
- sudo apt install coreutils
- sudo apt install expect
- sudo apt install libtool
- sudo apt install sed
- sudo apt install fuse
- sudo apt install net-tools
- sudo apt install python3
- sudo apt install python3-twisted
- sudo apt install build-essential
- sudo apt install devscripts
- sudo apt install equivs
- sudo apt install libssl-dev
- sudo apt install pkg-config
- sudo apt install libtasn1-dev
- sudo apt install gawk
- sudo apt install socat
- sudo apt install libseccomp-dev
- sudo apt install libfuse-dev
- sudo apt install autoconf-archive
- sudo apt install libcurl4-gnutls-dev
- sudo apt install libglib2.0-dev

Order of installation:

1) libtpms

   - git clone https://github.com/stefanberger/libtpms.git
   - cd libtpms
   - ./autogen.sh --with-tpm2 --with-openssl --prefix=/usr
   - make
   - make check
   - sudo make install
   
2) swtpm

   - git clone https://github.com/stefanberger/swtpm.git
   - cd swtpm
   - ./autogen.sh
   - ./configure --prefix=/usr --exec-prefix=/usr --with-cuse --with-openssl
   - make
   - make check
   - sudo make install
   
3) tpm2-tss

   - git clone https://github.com/tpm2-software/tpm2-tss.git
   - cd tpm2-tss
   - ./bootstrap
   - ./configure --disable-doxygen-doc
   - make
   - sudo make install

4) tpm2-tools

   - git clone https://github.com/tpm2-software/tpm2-tools.git
   - cd tpm2-tools
   - ./bootstrap
   - ./configure
   - make
   - sudo make install
   
Steps to instantiate TPM 2.0 emulator and run some test commands
----------------------------------------------------------------

1) Start the TPM 2.0 emulator

   - sudo swtpm_cuse --tpm2 -n tpm0 --tpmstate dir=/tmp --log file=/tmp/tpm0.log,level=99
   
   Check that the device "/dev/tpm0" has been created.
   
   - sudo chmod 777 /dev/tpm0
   
2) Initialize the TPM 2.0 emulator

   - sudo swtpm_ioctl --tpm-device /dev/tpm0 -i
   
3) Monitor TPM 2.0 messages/logs

   - sudo tail -f /tmp/tpm0.log
   
4) Send some test commands using tpm2-tools programs

   Set LD_LIBRARY_PATH, if needed
   - export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
   
   Send a startup command to TPM 2.0 device before sending any other command
   - tpm2_startup -T device --clear
   
   Run a command to create a primary object under Owner hierarchy
   - tpm2_createprimary -T device -C o -g sha256 -G ecc -c context.out
   (Refer tpm2-tools man page for more commands and their usage: https://github.com/tpm2-software/tpm2-tools/tree/master/man)
   
   See the transient handles and flush one or more handles for running subsequent commands (if needed) as only 3 handles can be stored inside of TPM 2.0 device
   - tpm2_getcap -T device handles-transient
   - tpm2_flushcontext -T device 0x80000000
   
5) Stop the TPM 2.0 emulator

   - sudo swtpm_ioctl --tpm-device /dev/tpm0 -s
   
   "/dev/tpm0" device will be removed with this command.

