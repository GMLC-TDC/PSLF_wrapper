# PSLF_wrapper

**\[DEVELOPERS NOTE\} This repository is based on a development version of PSLF and version 1.3 of HELICS.  It is not currently under development, though other efforts are ongoing to integrate PSLF and HELICS.  This code is made available as an example and for future reference if the effort is revived in the future.  The instruction on this page are also out of date for HELICS installation**

Positive Sequence Load Flow (PSLF) developed by GE is a widely used commercial tool for transmission level power system analysis by many power utilities and ISOs. Integration of PSLF with HELICS provides us with transmission level power flow, optimal power flow, and dynamic simulation capabilities.

GE currently supports the following features through it's python interface:
* Interface for power flow and dynamic data querying and modification.
* Interface for power flow calculation.
* Interface for basic dynamic simulation.

Currently not supported (and on GE’s to do list)
* Optimal Power Flow and LMP calculation.
* Load changing at each time step of the dynamic simulation.

PSLF wrapper implementation supports only steady flow simulation and dynamic power flow simulation will be integrated when the necessary API's are ready.

# Installation Instructions

PSLF tool is supported only in Windows so we will need to install HELICS on Windows. For steady state power flow simulation, we will also need GridLAB-D for distribution side simulation. The instructions that follow are for a multi-machine, multi-OS setup where PSLF integrated with HELICS runs on a Windows machine and GridLAB-D integrated with HELICS runs on a Linux machine.

## PSLF installation
  1. Install licensed version of PSLF on Windows

  2. Download the PLSF-HELICS integrated code.
  ```sh
        git clone https://github.com/GMLC-TDC/PSLF-wrapper.git
  ```

## HELICS installation on Windows

Follow the instructions for HELICS installation on Windows as per https://helics.readthedocs.io/en/latest/installation/windows.html. In addition to this, here are few steps that we need to take care of specific to PSLF integration. PSLF works with 32 bit version of python so we need to ensure that Microsoft Visual Studio, boost libraries, and python are 32 bit versions.

1. Install Boost Pre-built library 1.66 (32 version) needed by HELICS. Pre-built libraries are available in  https://dl.bintray.com/boostorg/release/1.66.0/binaries/. Please make sure “BOOST_INSTALL_PATH” environment variable is set to the location where boost is installed.

2. Install miniconda for python and swig packages. Download from https://repo.continuum.io/miniconda/. Select the option : "add to PATH env" so that installation path gets added to "PATH" enviroment variable. Next step is to install swig through miniconda.
```sh
    conda install swig
```

3. Download HELICS source code
```sh
    git clone https://github.com/GMLC-TDC/HELICS.git
```

4. We will have to build HELICS with python 2.7 support. Open "x86 Native Tools Command Prompt VS 2017" command prompt from Windows start menu.

```sh
    cd path/to/HELICS
    mkdir build
    cd build
    cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX="C:\local\helics-v1.3.0" -	DBOOST_ROOT="C:\local\boost_1_66_1" -DBUILD_PYTHON2_INTERFACE=ON -G "Visual Studio 15 2017" ..
    cmake --build . --config Release --target install
```

## HELICS installation on Linux

Please follow instructions in https://helics.readthedocs.io/en/latest/installation/linux.html to install HELICS on Linux. Add additional CMAKE flag during the configuration step.

```sh
    CMAKE_CXX_FLAGS = -fPIC -std=c++14
```

## GridLAB-D installation on Linux

1. Please download GridLABD.
```sh
    git clone -b feature/1024 http://github.com/gridlab-d/gridlab-d.git
```
2. Build/install third party prerequisites: xerces-c, automake, libtool
3. Build GridLAB-D:
    ```sh
        autoreconf -if
        ./configure --prefix=<install path> --with-helics=<helics install path> --enable-silent-rules ‘CFLAGS=-g -O0 -w’ ‘CXXFLAGS=-g -O0 -w -std=c++14’ ‘LDFLAGS=-g -O0 -w’
        make
        make install
    ```

## PSLF<->HELICS<->GridLAB-D Setup

PSLF integrated with HELICS should run on the Windows machine and HELICS broker and GridLAB-D integrated with HELICS should run on the Linux machine.

<img src="images/multi-machine.png">

If the windows machine is behind a firewall, then we need to open port range (23400 – 23700) to enable incoming traffic from Linux machine.

1. Add inbound rule for port range 23400 - 23700 on your windows machine

    a. Go to Control Panel->Windows Firewall ->Advanced settings ->Inbound Rules->New Rule

    b. Set Protocol type as TCP

    c. Set port range as 23400 – 23700

    d. Call IT helpdesk to open port range

    e. Restart computer for the changes get reflected

2. On the Windows machine, traverse to PSLF wrapper directory. Update the broker address and PSLF federate address in pslf_helics_config.json config file.

3. Specify the correct broker and PSLF federate addresses in pslf_helics_config.json. Start PSLF-HELICS to simulate steady state power flow simulation
    ```sh
        python pslf_wrapper.py
    ```

4. Start HELICS broker on Linux machine
    ```sh
        helics_broker 2 --log-level=3 --name=mainbroker --interface=tcp://<local IP>:23404
    ```

5. For this use case, we have used 700 bus transmission system through PSLF and a feeder model in GridLAB-D with detailed house appliances and HVACs. Follow instructions as described in https://github.com/GMLC-TDC/HELICS-Tutorial/tree/master/tutorials/1-DistributionFederation-ManualStart with few modifications.

    a. Since it is multi-machine setup, we need to provide broker address when starting the HELICS broker.

    ```sh
            helics_broker 2 --log-level=3 --name=mainbroker --interface=tcp://<local IP>:23404
    ```

    b. Modify the "core_init_string" in https://github.com/GMLC-TDC/HELICS-Tutorial/blob/master/test_system_data/gldFeeders/B2/G_1/DistributionSim_B2_G_1.json

	```sh
        “--federates=1 --broker_address=tcp://<broker ip> --interface=tcp://<local ip>”
    ```
This ensures that both PSLF and GridLAB-D example connect to the same broker address.

## Release
PSLF-Wrapper code is distributed under the terms of the BSD-3 clause license. All new
contributions must be made under this license. [LICENSE](LICENSE)

SPDX-License-Identifier: BSD-3-Clause
