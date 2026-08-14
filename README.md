# Network Flow Watermarking for Resynchronization Based on Temporal Reconstruction

This project provides a four-node implementation of a network flow watermarking method based on temporal reconstruction and resynchronization. The method reconstructs the sender-side temporal reference from TCP Timestamp Values (TSval) and combines a fixed synchronization word, dynamic window alignment, soft tail-biting convolutional decoding, and CRC verification to embed, synchronize, and recover watermarks.



## 1. Node Roles

### 1.1 Node A

Node A is the traffic sender and watermark embedder. Its main functions are:

1. generate and transmit TCP flows;
2. apply CRC and tail-biting convolutional coding to the watermark payload;
3. prepend the fixed synchronization field and map the frame to temporal windows;
4. embed the watermark by selectively delaying TCP packets.

### 1.2 Node B

Node B is the controlled-channel node. It forwards traffic and can apply disturbances such as delay, packet loss, and TCP resegmentation according to a supplied trajectory.

### 1.3 Node C

Node C is the NAT and observation node. It provides IPv4 forwarding, DNAT/MASQUERADE rule construction, and ingress/egress packet capture.

### 1.4 Node D

Node D is the receiver and detector. Its main functions are:

1. receive TCP flows and parse PCAP files;
2. select the target flow and handle retransmitted or duplicate segments;
3. reconstruct the temporal reference from TSval;
4. search synchronization candidates and perform dynamic window alignment;
5. extract soft metrics and perform soft tail-biting convolutional decoding;
6. verify the recovered watermark with CRC and output the recovery result.

## 2. Core Method

The main processing flow is:

1. append a CRC to the watermark payload and apply tail-biting convolutional encoding;
2. prepend a fixed synchronization field and map logical bits to temporal windows;
3. embed the watermark at Node A through bounded packet delays;
4. forward the traffic through the controlled channel at Node B and the NAT at Node C;
5. reconstruct the temporal reference from TSval at Node D, then perform synchronization, dynamic alignment, soft decoding, and CRC verification.

The protocol described in the paper uses a 12-bit payload, an 8-bit CRC, a rate-1/2 tail-biting convolutional code, the fixed synchronization word `00010110`, 48 logical positions, and the sparse temporal mapping `0 -> 001` and `1 -> 010`, resulting in 144 temporal windows.

## 3. Code Structure

```text
algorithm-package/
├── README.md                 Project overview
├── requirements.txt         Python dependencies
├── src/
│   ├── paper_pipeline.py    Index of paper algorithm steps
│   ├── common/              Shared I/O and TCP endpoint functions
│   ├── node_A/              Traffic generation, coding, and watermark embedding
│   ├── node_B/              Controlled channel, disturbances, and forwarding
│   ├── node_C/              NAT forwarding and packet capture
│   └── node_D/              Reception, reconstruction, synchronization, decoding, and detection
└── tests/                    Core algorithm and node-interface tests
```

Each node provides a simplified public interface through `running.py`. The main algorithms are located in the `code/` and `embed/` directories of Node A and the `reconstruct/`, `synchronize/`, `decode/`, and `detect/` directories of Node D.

## 4. Installation and Tests

Install the dependencies:

```bash
python -m pip install -r requirements.txt
```

Run the tests:

```bash
python -m pytest tests -q
```

`NetfilterQueue` is required only for live NFQUEUE packet processing on Linux. Offline frame coding and receiver decoding primarily require NumPy.


## E-mail
If you have any question, please feel free to contact us by e-mail (jiangtaozhai@nuist.edu.cn).

