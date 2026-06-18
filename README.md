# Proxmark3 X dual frequency key cloning guide

Guide to use the Proxmark3 X to clone a dual-frequency Schlage 9691T to a dual-frequency (125kHz, 13.56 MHz Gen1a/Gen4 magic) 1K card. This guide is only intended for personal or educational use. Do not copy keys without explicit permission from their owner.

## Hardware

Original fob: Schlage 9691T
Tool: [Proxmark3 X](https://shop.mtoolstec.com/product/proxmark3-x), connected via USB-C to a Windows 11 computer.
Blank card: [Dual frequency 125kHz/13.56MHz 1K card](https://www.amazon.com/dp/B07MJ7JNVV)

The card used is a MIFARE Classic 1k hybrid Gen 1a and Gen 4 GDM card. The Gen 1a backdoor must be disabled for a higher security card reader to accept the card. For readers that check for the device signature, a 4k card will need to be used.

## Software Setup

There are 2 sets of switches on the right hand side of the Proxmark3 X, one upper and one lower. The upper switch can be set to 充电 (charge), 关 (off), and ID, and the lower switch, labeled 线圈 (coil), can be set to PM3 and ID. For this guide, the top switch was set to 关 (off), the middle position, and the bottom switch was set to PM3, the top position. Connect the Proxmark 3 X to a Windows computer using a USB-C cable.

### Using precompiled build

Download and decompress the precompiled build for RRG / Iceman [here](https://www.proxmarkbuilds.org/) (Select Latest RRG / Iceman generic build for Proxmark3 devices (non RDV4), for Proxmark3 Easy, RDV1, RDV2, RDV3, etc etc).

Run `pm3-flash-all.bat`. When that finishes, run `pm3.bat` and verify that it detects the ProxMark.

### Compiling ProxSpace if precompiled build is not available

Download the latest release of [ProxSpace](https://github.com/Gator96100/ProxSpace) and extract it to the desired folder. Make sure that there are no spaces in the file path to the program. Then, launch `runme64.bat` and let it detect the device and begin setup. It may give an error about a firmware mismatch, but this will be fixed.

#### Clone the Iceman repository:

```bash
git clone https://github.com/RfidResearchGroup/proxmark3.git
cd proxmark3
```

#### Configure the hardware profile for the ProxMark3 X

Copy the platform configuration file from the included sample file.

```bash
cp Makefile.platform.sample Makefile.platform
```

Open the newly made `Makefile.platform`:

```bash
nano Makefile.platform
```

Find the following 2 lines and uncomment them by removing the `#` at the start of the line:

```bash
PLATFORM=PM3GENERIC
PLATFORM_EXTRAS=BTADDON
```

Save and exit (in Nano, press Ctrl+O, Enter, then Ctrl+X)

#### Compile the source code

Compile the environment with:

```bash
make clean
make -j
```

#### Flash the firmware

Flash the firmware with:

```bash
./pm3-flash-all
```

If the script doesn't correctly autodetect the COM port of the ProxMark3 X, explicitly pass in the port number. This can be obtained by opening Device Manager and opening the Ports dropdown.

```bash
./pm3-flash-all COM3
```

#### Launch the client

After compiling, make sure that the current folder is `proxmark3`, then run

```bash
./pm3
```

The hardware can be verified and tuned with:

```bash
hw tune
```

The Proxmark3 X should now be connected.

If any of the commands below are run and return a `[!] could not create file` or `[!] file not found or locked` error, this is likely because the ProxSpace installation path was moved after it was originally installed. This can be changed to the current directory with:

```bash
prefs set savepaths --def .
prefs set savepaths --dump .
prefs set savepaths --trace .
```

## Clone the 125 kHz

### Copy the data from the original card

Place the original fob onto the low frequency antenna of the Proxmark3, then run:

```bash
lf search
```

It should output something like (sensitive information replaced with X):

```bash
[=] Note: False Positives ARE possible
[=]
[=] Checking for known tags...
[=]
[+] [H10301  ] HID H10301 26-bit                FC: XXX  CN: XXX  parity ( ok )
[+] [ind26   ] Indala 26-bit                    FC: XXXX  CN: XXX  parity ( ok )
[=] found 2 matching 26-bit formats

[=] Trying with a preamble bit...
[+] [ind27   ] Indala 27-bit                    FC: XXXX  CN: XXX
[+] [indasc27] Indala ASC 27-bit                FC: XXX  CN: XXXX
[+] [Tecom27 ] Tecom 27-bit                     FC: XXX  CN: XXXXX
[=] found 3 matching 27-bit formats

[=] raw: xxxxxxxxxxxxxxxxxxxxxxxx

[+] Valid HID Prox ID found!

[=] Searching for auth LF and special cases...
[=] Couldn't identify a chipset
```

Note the `raw:` field. Copy this data for the next section.

### Clone data to the blank

Put the blank card on the low frequency antenna and verify that it can be read:

```bash
lf search
```

This will probably output something like:

```bash
[=] Note: False Positives ARE possible
[=]
[=] Checking for known tags...
[=]
[+] EM 410x ID XXXXXXXXXX
[+] EM410x ( RF/64 )
[=] -------- Possible de-scramble patterns ---------
[+] Unique TAG ID      : XXXXXXXXXX
[=] HoneyWell IdentKey
[+]     DEZ 8          : XXXXXXXX
[+]     DEZ 10         : XXXXXXXXXX
[+]     DEZ 5.5        : XXXXX.XXXXX
[+]     DEZ 3.5A       : XXX.XXXXX
[+]     DEZ 3.5B       : XXX.XXXXX
[+]     DEZ 3.5C       : XXX.XXXXX
[+]     DEZ 14/IK2     : XXXXXXXXXXXXXX
[+]     DEZ 15/IK3     : XXXXXXXXXXXXXXX
[+]     DEZ 20/ZK      : XXXXXXXXXXXXXXXXXXXX
[=]
[+] Other              : XXXXX_XXX_XXXXXXXX
[+] Pattern Paxton     : XXXXXXX [0xXXXXXX]
[+] Pattern 1          : XXXXX [0xXXXX]
[+] Pattern Sebury     : XXXX X XXXX  [0xXXXX 0xX 0xXXXX]
[+] VD / ID            : XXX / XXXXXXXXXX
[+] Pattern ELECTRA    : X XXXX
[=] ------------------------------------------------

[+] Valid EM410x ID found!

[=] Searching for auth LF and special cases...
[=] Couldn't identify a chipset
```

Next, run the following, replacing `xxxxxxxxxxxxxxxxxxxxxxxx` with the data after `raw:` from the previous section.

```bash
lf em clone --id xxxxxxxxxxxxxxxxxxxxxxxx
```

This should output:

```bash
[=] Preparing to clone HID tag using raw xxxxxxxxxxxxxxxxxxxxxxxx
[+] Done!
[?] Hint: Try `lf hid reader` to verify
```

Verify by running `lf search` again with the clone on the low frequency antenna. This output should show that the data on the card is functionally identical to the original fob.

```bash
[=] Note: False Positives ARE possible
[=]
[=] Checking for known tags...
[=]
[+] [H10301  ] HID H10301 26-bit                FC: XXX  CN: XXX  parity ( ok )
[+] [ind26   ] Indala 26-bit                    FC: XXXX  CN: XXX  parity ( ok )
[=] found 2 matching 26-bit formats

[=] Trying with a preamble bit...
[+] [ind27   ] Indala 27-bit                    FC: XXXX  CN: XXX
[+] [indasc27] Indala ASC 27-bit                FC: XXX  CN: XXXX
[+] [Tecom27 ] Tecom 27-bit                     FC: XXX  CN: XXXXX
[=] found 3 matching 27-bit formats

[=] raw: xxxxxxxxxxxxxxxxxxxxxxxx

[+] Valid HID Prox ID found!

[=] Searching for auth LF and special cases...
[=] Couldn't identify a chipset
```

The 125 kHz chip on the card should now be programmed!

## Clone the 13.56 MHz

### Copy the data from the original card

Place the original fob onto the high frequency antenna of the Proxmark3, then run:

```bash
hf search
```

It should output something like (sensitive information replaced with X):

```bash
[/] Searching for ISO14443-A tag...
[=] ---------- ISO14443-A Information ----------
[+]  UID: XX XX XX XX   ( ONUID, re-used )
[+] ATQA: XX XX
[+]  SAK: XX [X]
[+] Possible types:
[+]    MIFARE Classic 1K
[=]
[=] Proprietary non iso14443-4 card found
[=] RATS not supported
[+] Prng detection..... hard
[=]  IC signature public key name: NXP MIFARE Classic MFC1C14_x
[=] IC signature public key value: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
[=]                              : XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
[=]                              : XX
[=]     Elliptic curve parameters: xxxxxxxxx
[=]              TAG IC Signature: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
[=]                              : XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
[+]        Signature verification: successful

[?] Hint: Try `hf mf info`


[+] Valid ISO 14443-A tag found
```

Run:

```bash
hf mf autopwn
```

This may take a few minutes while the program runs, but it should eventually create three files: `hf-mf-XXXXXXXX-dump.bin`, `hf-mf-XXXXXXXX-key.bin`, and `hf-mf-XXXXXXXX-dump.json`.

### Clone data to the blank

Put the blank card on the high frequency antenna and run:

```bash
hf search
```

This should output something like (sensitive information replaced with X):

```bash
[-] Searching for ISO14443-A tag...
[=] ---------- ISO14443-A Information ----------
[+]  UID: XX XX XX XX   ( ONUID, re-used )
[+] ATQA: XX XX
[+]  SAK: XX [X]
[+] Possible types:
[+]    MIFARE Classic 1K
[=]
[=] Proprietary non iso14443-4 card found
[=] RATS not supported

[+] Magic capabilities... Gen 1a
[+] Magic capabilities... Gen 4 GDM / USCUID ( ZUID Gen1 Magic Wakeup )
[+] Prng detection..... weak

[?] Hint: Use `hf mf c*` magic commands
[?] Hint: Use `hf mf gdm* --gen1a` magic commands
[?] Hint: Try `hf mf info`


[+] Valid ISO 14443-A tag found
```

To copy the data over, run:

```bash
hf mf cload -f hf-mf-XXXXXXXX-dump.bin
```

### Verify that the clone was successful

Copy the dump from the clone card:

```bash
hf mf csave -f verify.bin
```

This can be compared with the original `hf-mf-XXXXXXXX-dump.bin` with any terminal tools of choice but either can be visually inspected with:

```bash
hf mf view -f verify.bin
```

or 

```bash
hf mf view -f hf-mf-XXXXXXXX-dump.bin
```

If they match, then the clone was successful.

### Disable Magic Gen 1 backdoor

Many readers will not accept a Gen 1a magic card and they can easily check this by sending a backdoor sequence to the card. Luckily, the blank card chosen is a hybrid Gen 1a and Gen 4 GDM card, which means the internal configuration registers can be programmed to disable the Gen 1a backdoor.

First check the current configuration by running

```bash
hf mf gdmcfg --gen1a
```

This should output something like:

```bash
[+] ------------------- GDM Gen4 Configuration -----------------------------------------
[+] 7AFF0000000000000000000000000008
[+] 7AFF............................ Magic wakeup enabled with GDM cfg block access
[+] ....00.......................... Magic wakeup style Gen1a 40(7)/43
[+] ......000000.................... unknown
[+] ............00.................. Key B use allowed when readable by ACL
[+] ..............00................ CUID Disabled
[+] ................00.............. n/a
[+] ..................00............ MFC EV1 perso. 4B UID from Block 0
[+] ....................00.......... Shadow mode disabled
[+] ......................00........ Magic auth disabled
[+] ........................00...... Static encrypted nonce disabled
[+] ..........................00.... MFC EV1 signature disabled
[+] ............................00.. n/a
[+] ..............................08 SAK
```

For most of these registers, `00` is disabled and `5A` is enabled. To disable magic wakeup, run

```bash
[usb] pm3 --> hf mf gdmsetcfg --gen1a -d 85000000000000000000000000000008
```

and verify the configuration with:

```bash
hf mf gdmcfg --gen1a
```

It should now look like the following, with Magic wakeup disabled (8500).

```bash
[+] ------------------- GDM Gen4 Configuration -----------------------------------------
[+] 85000000000000000000000000000008
[+] 8500............................ Magic wakeup disabled
[+] ....00.......................... Magic wakeup style Gen1a 40(7)/43
[+] ......000000.................... unknown
[+] ............00.................. Key B use allowed when readable by ACL
[+] ..............00................ CUID Disabled
[+] ................00.............. n/a
[+] ..................00............ MFC EV1 perso. 4B UID from Block 0
[+] ....................00.......... Shadow mode disabled
[+] ......................00........ Magic auth disabled
[+] ........................00...... Static encrypted nonce disabled
[+] ..........................00.... MFC EV1 signature disabled
[+] ............................00.. n/a
[+] ..............................08 SAK
```

The 13.56 MHz chip on the card should now be programmed!

## References
https://github.com/Gator96100/ProxSpace
https://proxmark3x.mtoolstec.com/guides/use-proxmark3-x-on-windows
