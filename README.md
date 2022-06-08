# Donar

Donar enables anonymous VoIP with good quality-of-experience (QoE) over the Tor network. No individual Tor link can match VoIP networking requirements. Donar bridges this gap by spreading VoIP traffic over several links.

## 1) Download build+runtime dependencies

Instructions are provided for Debian 11 "bullseye" (current Debian stable a:

```bash
sudo apt update
sudo apt-get install -y \
  build-essential \
  cmake \
  libglib2.0-0 \
  libgstreamer1.0 \
  libgstreamer-plugins-bad1.0-dev \
  gstreamer1.0-alsa \
  gstreamer1.0-plugins-bad \
  gstreamer1.0-plugins-good \
  gstreamer1.0-plugins-ugly \
  gstreamer1.0-plugins-rtp \
  gstreamer1.0-pulseaudio \
  libevent-2.1 \
  zlib1g \
  libssl1.1 \
  zstd \
  liblzma5 \
  git \
  tor
```

## 2) Compile binaries

Download this repository:

```
git clone https://github.com/superboum/Donar.git
cd Donar
```

Compile the binaries:

```
mkdir target ; cd target
cmake ..
make -j8
```

## 3) Callee

In a first terminal:

```bash
touch /tmp/empty
tor \
  -f /tmp/empty \
  --defaults-torrc /tmp/empty \
  --hush \
  --UseEntryGuards 1 \
  --NumEntryGuards 2 \
  --NumPrimaryGuards 2 \
  --SocksPort 0 \
  --ControlPort 9100 \
  --DataDirectory /tmp/tor-callee
```

In a second terminal:

```bash
./donar -s -a lightning -l 12 -p 'fast_count=3!tick_tock=0!measlat=0!window=2000' -e 5000 -r 5000
```

In a third terminal:

```bash
./dcall 127.13.3.7
```

Your "address" is contained inside the `onion_services.pub`, you must transmit it out of band to people that want to call you.

## 4) Caller

In a first terminal:

```bash
touch /tmp/empty
/usr/local/bin/tor3 \
  -f /tmp/empty \
  --defaults-torrc /tmp/empty \
  --hush \
  --UseEntryGuards 1 \
  --NumEntryGuards 2 \
  --NumPrimaryGuards 2 \
  --SocksPort "127.0.0.1:9000 IsolateDestPort IsolateDestAddr IsolateClientAddr" \
  --DataDirectory /tmp/tor-caller
```

In a second terminal:

```bash
./donar -c -o onion_services.pub -a lightning -l 12 -p 'fast_count=3!tick_tock=0!window=2000' -e 5000 -r 5000
```

In a third terminal:

```bash
./dcall 127.13.3.7
```

## 5) Other configurations and tools

Get a look at the `scripts/2021/` folder, and start by the file `donardup`.
In this folder, you will find the files that I have used to automate my latest tests.

## 6) Test with 2-relay circuits

To test with 2-relay circuits instead of 3-relay ones, you must compile your own patched version of Tor.
The only modification you need is to set the `DEFAULT_ROUTE_LEN` constant to `2` instead of `3`.
Currently, this constant is here:

```
./src/core/or/or.h:894:#define DEFAULT_ROUTE_LEN 3
```

