# ONVIF Uplink Camera Plugin for VXG Cloud

Learn more about <a href="https://www.videoexpertsgroup.com">Cloud Video Surveillance</a>\
Learn more about usage and development in our <a href="https://help.videoexpertsgroup.com/kb/camera-uplink-agent">Knowledge Base</a>

The `ONVIF Uplink Camera Plugin` is a simple `C++` reference code for integration of `IP cameras` with `VXG Cloud`.
<br>
<br>
This library requires requires libwebsockets version 4.0 (4.0.20-2) as a minimum.
<br>


## CMake Guide

### Compile Application for Local Machine
1. Install necessary libraries.
```
sudo apt update && sudo apt install -y build-essential libwebsockets-dev cmake
```
2. Setup build and install directories.
```
cd <SRC_DIRECTORY>
mkdir build && mkdir install
```
3. Build.
```
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=../install
make && make install
```
Binary executable will be found at `<SRC_DIRECTORY>/build/vxg-proxy-client`
Library files will be found at `<SRC_DIRECTORY>/install`


### Cross Compile Application
<b>This process may require the cross compilation of additional libraries if they are not already built into the toolchain of the target platform.</b>\
For example: If the target toolchain may have libssl and libcrypto already built, but not libwebsockets (LWS). You will need to cross-compile LWS with SSL support before cross compiling the Uplink App.

1. Install necessary build libraries on host machine.
```
sudo apt update && sudo apt install -y build-essential cmake
```
2. Setup build and install directories.
```
cd <SRC_DIRECTORY>
mkdir build && mkdir install
```
3. Prepare cross file.
Example: `example_cross.cmake`
```
# the name of the target operating system
set(CMAKE_SYSTEM_NAME Linux)

# which compilers to use for C and C++
set(CMAKE_C_COMPILER   <TARGET>-gcc)
set(CMAKE_CXX_COMPILER <TARGET>-g++)

# where is the target environment located
set(CMAKE_FIND_ROOT_PATH  <PATH_TO_TOOLCHAIN_ROOT>)

# adjust the default behavior of the FIND_XXX() commands:
# search programs in the host environment
set(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)

# search headers and libraries in the target environment
set(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
set(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY)

set(CMAKE_C_FLAGS "-O2 -g -Wall -fmessage-length=0")
set(CMAKE_CXX_FLAGS "-O2 -g -Wall -fmessage-length=0")
```
4. Cross compile.
```
cd build
cmake .. -DCMAKE_INSTALL_PREFIX="../install" -DCMAKE_TOOLCHAIN_FILE="../cross/example_cross.cmake"
# Optional -DLWS_LIB <Path to target platform libwebsockets library>
#     -DLWS_INCLUDE <Path to target platform libwebsockets include directory>
#     -DSSL_LIB <Path to target platform OpenSSL library file>
#     -DCRYPTO_LIB <Path to target platform OpenSSL library file>
#     -DSSL_INCLUDE <Path to target platform OpenSSL include directory>
make && make install
```
Binary executable will be found at `<SRC_DIRECTORY>/build/vxg-proxy-client`\
Library files will be found at `<SRC_DIRECTORY>/install`

## Example Application Usage

### Commandline Options
```
--token <AUTH_TOKEN> (Access token for camera, generated by VMS)
--serial <DEVICE_SERIAL> (Serial number of device)
--http (disable HTTPS)
--insecure (do not verify HTTPS certificates)
--debug (enable debug logs)
-f <FWD_NAME>:http|https|tcp:[<IPV4_ADDRESS|HOSTNAME|IPV6_ADDRESS>]:<TARGET_PORT>
```

### Environment Variables
`VXG_API_HOST` - hostname of VXG Camera API (by default: camera.vxg.io)

`VXG_API_PATH` - template for VXG Camera API token request path + query string
(by default: /v1/token?serial_id=%s, %s here will be auto-replaced with device serial number during runtime)

`VXG_API_PASSWORD` - API password/authorization
(MUST be set if no --token argument provided)

`PROXY_API_PATH` - URI path for requesting websocket endpoint from proxy API
(by default: /api/ws-endpoint)

`AUTH_TOKEN` - so you could also provide auth token via env variable

`DEVICE_SERIAL` - so you could also provide device serial number via env variable

### Running Application
There are two main options for executing the sample application.

#### Using Provisioning Server to Retireve Access Token
This is the prefered method in practice. Access token will need to be created by creating an Uplink Camera on VMS using the Camera's Serial Number and MAC Address.
```
VXG_API_PASSWORD=<MAC_ADDRESS> ./vxg-proxy-client --serial <SERIAL_NUMBER> -f fwd1:http:[127.0.0.1]:80 -f fwd2:tcp:[127.0.0.1]:554
```
`fwd1:http:[127.0.0.1]:80` creates a forward for port 80 on the local machine. Used for providing access to camera Web UI.\
`fwd2:tcp:[127.0.0.1]:554` creates a forward for port 554 on the local machine. Used for providing access to camera RTSP stream.

#### Using Access Token from VMS
This method is mainly used for testing application. Token will need to be created and fetched from VMS manually.
```
./vxg-proxy-client --token eyJjYW1pZCI6IDEyMzQ1NiwgInVwbGluayI6ICJkZXYtYXBpLnByb3h5LmNsb3VkLXZtcy5jb20ifQo= -f fwd1:http:[127.0.0.1]:80 -f fwd2:tcp:[127.0.0.1]:554
```
This method involves manually providing the access token to the application. The application will then avoid requesting the token from the provisioning server using `DEVICE_SERIAL` and `VXG_API_PASSWORD`\
`fwd1:http:[127.0.0.1]:80` creates a forward for port 80 on the local machine. Used for providing access to camera Web UI.\
`fwd2:tcp:[127.0.0.1]:554` creates a forward for port 554 on the local machine. Used for providing access to camera RTSP stream.\

### Checking Connection After Running Application
You can request connection info for a specific camera using the Uplink Server's API.
```
curl -H 'Authorization: Bearer <PROXY_API_TOKEN>' https://<PROXY_API_HOSTNAME>/api/device/<DEVICE_ID>
```
`DEVICE_ID` can be retrieved from the Access Token used during execution of application.

Example response:
```
{
     "camid": "123456",
     "proxy_id": "f2f8b6edd7ebbdbd4100",
     "forwards": [
         {
            "name": "fwd1",
            "protocol": "tcp",
            "host": "google.com",
            "port": 443,
            "proxy_port": 38103,
            "proxy_host": "beeeaacf8af994911700.proxy.cloud-vms.com"
         }
     ]
}
```
`proxy_host`:`proxy_port` is the address that you can access the forwarded address and port from.
