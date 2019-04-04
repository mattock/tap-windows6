TAP-Windows driver (NDIS 6)
===========================

This is an NDIS 6.20/6.30 implementation of the TAP-Windows driver, used by
OpenVPN and other apps. NDIS 6.20 drivers can run on Windows 7 or higher except
on ARM64 desktop systems where, since the platform relies on next-gen power
management in its drivers, NDIS 6.30 is required.

Build
-----

To build, the following prerequisites are required:

- Python 2.7
- Microsoft Windows 10 EWDK (Enterprise Windows Driver Kit)
    - Visual Studio+Windows Driver Kit works too. Make sure to work from a
      "Command Prompt for Visual Studio" and to call buildtap.py with "--sdk=wdk".
- Source code directory of **devcon** sample from WDK (optional)
    - https://github.com/Microsoft/Windows-driver-samples/ setup/devcon
- Windows code signing certificate
- Git (not strictly required, but useful for running commands using bundled bash shell)
- MakeNSIS (optional)
- Prebuilt tapinstall.exe binaries (optional)

Make sure you add Python's install directory (usually c:\\python27) to the PATH 
environment variable.

These instructions have been tested on Windows 10 using Windows' CMD.exe.

View build script options::

  $ python buildtap.py
  Usage: buildtap.py [options]
  
  Options:
    -h, --help           show this help message and exit
    -s SRC, --src=SRC    TAP-Windows top-level directory, default=<CWD>
    --ti=TAPINSTALL      tapinstall (i.e. devcon) directory (optional)
    -d, --debug          enable debug build
    -c, --clean          do an nmake clean before build
    -b, --build          build TAP-Windows and possibly tapinstall (add -c to
                         clean before build)
    --sdk=SDK            SDK to use for building: ewdk or wdk, default=ewdk
    --sign               sign the driver files
    -p, --package        generate an NSIS installer from the compiled files
    --cert=CERT          Common name of code signing certificate,
                         default=openvpn
    --certfile=CERTFILE  Path to the code signing certificate
    --certpw=CERTPW      Password for the code signing certificate/key
                         (optional)
    --crosscert=CERT     The cross-certificate file to use, default=MSCV-
                         VSClass3.cer
    --timestamp=URL      Timestamp URL to use, default=http://timestamp.verisign
                         .com/scripts/timstamp.dll
    -a, --oas            Build for OpenVPN Access Server clients

Edit **version.m4** and **paths.py** as necessary then build::

  $ python buildtap.py -b

On successful completion, all build products will be placed in the "dist" 
directory as well as tap6.tar.gz. The NSIS installer package will be placed to
the build root directory.

Building tapinstall (optional)
------------------------------

The easiest way to build tapinstall is to clone the Microsoft driver samples
and copy the source for devcon.exe into the tap-windows6 tree. Using PowerShell:

    git clone https://github.com/Microsoft/Windows-driver-samples
    Copy-Item -Recurse Windows-driver-samples/setup/devcon tap-windows6
    cd tap-windows6
    python.exe buildtap.py -b --ti=devcon

The build system also supports reuse of pre-built executables. To make sure the
buildsystem finds the executables, create the following directory structure
under tap-windows6 directory:
::
  tapinstall
  ├── Release
  │   └── devcon.exe
  ├── x64
  │   └── Release
  │       └── devcon.exe
  └── ARM64
      └── Release
          └── devcon.exe

This structure is equal to what building tapinstall would create. Call
buildtap.py with "--ti=tapinstall".

Please note that the NSIS packaging (-p) step will fail if you don't have
tapinstall.exe available. Also don't use the "-c" flag or the above directories
will get wiped before MakeNSIS is able to find them.

Install/Update/Remove
---------------------

The driver can be installed using a command-line tool, tapinstall.exe, which is
bundled with OpenVPN and tap-windows installers. Note that in some versions of
OpenVPN tapinstall.exe is called devcon.exe. To install, update or remove the
tap-windows NDIS 6 driver follow these steps:

- place tapinstall.exe/devcon.exe to your PATH
- open an Administrator shell
- cd to **dist**
- cd to **amd64**, **i386**, or **arm64** depending on your system's processor architecture.

Install::

  $ tapinstall install OemVista.inf TAP0901

Update::

  $ tapinstall update OemVista.inf TAP0901

Remove::

  $ tapinstall remove TAP0901

Notes on proxies
----------------

It is possible to build tap-windows6 without connectivity to the Internet but 
any attempt to timestamp the driver will fail. For this reason configure your 
outbound proxy server before starting the build. Note that the command prompt 
also needs to be restarted to make use of new proxy settings.

About driver signing
--------------------

While tap-windows6 buildsystem supports driver signing, it is unlikely you can
actually make much use of it. This is because driver signing requirements in
recent Windows versions have become more and more strict. Nowadays you will need
to use a combination of cross-signing with an EV dongle
(Windows 7/8/8.1/Server 2012r2), attestation signing (Windows 10) and
WHQL-signing based on succesful HLK test result submission
(Windows Server 2016/2019).

In addition making older Windows versions such as Vista play along with
signatures that Windows 7/8/8.1/Server 2012r2 accepts can be rather challenging.
A good starting point on this topic is the
`building tap-windows6 <https://community.openvpn.net/openvpn/wiki/BuildingTapWindows6>`_
page on the OpenVPN community wiki. As that page points out, having two
completely separate Authenticode signatures may be the only reasonable option.
Fortunately there is a tool, `Sign-Tap6 <https://github.com/mattock/sign-tap6/>`_,
which can be used to append secondary signatures to the tap-windows6 driver or
to handle the entire signing process if necessary.

The Sign-tap6 tool can also be used for signing tap-windows6 driver files using
and EV dongle, which is the way to do signatures for Windows 7/8/8.1 and Server
2012r2.

License
-------

See the file `COPYING <COPYING>`_.
