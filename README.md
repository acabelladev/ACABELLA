
# ACABELLA

[![Python 3.10](https://img.shields.io/badge/python-3.10-blue.svg)](https://www.python.org/downloads/release/python-3100/)

ACABELLA is a tool for analyzing the security of Attribute-based encryption (ABE) schemes.

ABE is a type of public key encryption in which the keys are linked to attributes. It enforces access control on a cryptographic level in a fine-grained fashion. Both ABE and its multi-authority (MA) variant, MA-ABE can be used in the Cloud and in medical environments to protect private data.

ACABELLA analyzes the security of an ABE scheme according to:

    - The Venema-Alpar cryptanalysis framework [1]
    - The AC17 framework [2]
    - The FABEO property [3]

ACABELLA also looks for master-key and decryption attacks in the provided ABE scheme taking
into account corruption.

The web version of ACABELLA is deployed at [https://www.acabel.la](https://www.acabel.la).

## Requirements

- For ACABELLA:
    * `Python` 3.10.6
    * `sympy` 1.11.1
  
- For the web application:
    * `click` 8.0.3
    * `Flask` 2.2.0
    * `Werkzeug` 2.2.2

- For running the tests:
    * `pytest` 7.1.2
 
## Using ACABELLA

The ACABELLA tool is available to be used in three ways: directly in Python,
or using one of the two tools that rely on the implementation of ACABELLA in
Python `acabella_cmd` and `acabella_web`.

### The ACABELLA API

The API provides the following classes that can be used to analyze
the security of ABE schemes:

- The `Attack` class, which independently provide methods for finding
master key and decryption attacks and analyze the security of an ABE
scheme.
- The `Analysis` class, which prepares batches of attacks and run them
based on the description of an ABE scheme. It is also possible to use
the ACABELLA JSON format, which describes the properties of an
ABE scheme and particular corruption environments where security is
analyzed.

We refer the user to the `test/` directory where many examples are provided. Particularly, the example attacks rely on the `Attack` class whereas the `analysis` directory rely on the ACABELLA `Analysis` class. The interested user can also inspect the source code of the `acabella_cmd` and `acabella_web` tools which heavily rely on the `Analysis` class.

### `acabella_cmd`: The ACABELLA analysis command line tool

`acabella_cmd` receives a JSON input describing an ABE scheme and analyzes its security
according to the parameters passed.

```bash
[*] ACABELLA cmd tool
usage: acabella_cmd.py [-h] -a {mk,da,sec,cond,all} -c CONFIG

options:
  -h, --help            show this help message and exit
  -a {mk,da,sec,cond,all}, --analysis {mk,da,sec,cond,all}
                        Select the type of analysis to perform: mk for master key attack, da for decryption attack, ac17 for security analysis, cond for conditional attack and all for
                        performing every analysis type
  -c CONFIG, --config CONFIG
                        Configuration file for the analysis type in ACABELLA JSON format
```

`acabella_cmd` is located at `ACABELLA/tools/acabella_cmd`.


For instance, to look for master-key attacks in the YJ12 [4] MA-ABE scheme, we can use the following JSON input file:

```json
{
    "scheme_id": "yj12",
    "analysis": "master_key",
    "k": ["r * b / bp + alpha_i/bp", "r"],
    "master_key": "alpha_i",
    "unknown_vars" :  ["alpha_i", "r"],
    "corruption_model" : "mixed_AA_corr",
    "corruptable_vars": [ { "type":"MPK_AA", "var":"b/bp" }],
    "MPK_CA": ["r"],
    "MPK_AA": ["alpha_i", "b/bp"],
    "MPK_vars": [],
    "GP_vars": ["b", "bp"]
}
```

An example utilization in this case would be:

```bash
$ python acabella_cmd.py -a mk -c examples/yj12/yj12_config_mka_corruption.json

[*] ACABELLA cmd tool


[*] Analyzing scheme...



[*] Master key attack results:


List of encodings:
	 k0[i] : alpha_i/bp + b*r/bp
	 k1[i] : r
	 gp : b
	 gp : bp

For the corruption of an attribute authority AA[j] and attacking an attribute authority AA[i].

Information on additional encodings:
	[*] b corresponds to gp0
	[*] bp corresponds to gp1

NOTE: Global parameters gp[i] are added to the matrix as key encodings and could appear as k[i] elements.

Structure of CA/AAs:
	Contents of the CA MPK encodings: [r]
	Contents of the AA MPK encodings: [alpha_i, b/bp]

List of variables obtained via the corruption of AA[j]:
	b/bp from MPK_AA



[*] Master key attack with corruption found: bp*k0[i] + -b*k1[i]
```

The interested user could find many example inputs in JSON format at `ACABELLA/tools/acabella_cmd\examples`.

### `acabella_web`: The ACABELLA web application

It provides a web interface for analyzing the security of ABE schemes.
It is located at tools/acabella_web and uses flask. Also note that the requirements listed at `tools/acabella_web/requirements.txt` must be installed.

It can be deployed locally with the following command:

```bash
$ flask --app flaskr --debug run
```

Finally, the web version of ACABELLA is deployed at [https://www.acabel.la](https://www.acabel.la).

![image info](img/acabella.png)


### Other tools

ACABELLA provides other tools to transform the JSON ACABELLA inputs into inputs for the ABGW17 [2] tool.

#### ABGW bridge

It translates the ACABELLA format for describing ABE schemes into a valid
input to the ABGW tool.

In [2], the authors provided a tool for analyzing the security    
of ABE schemes within the context of the group generic model. In ACABELLA,    
we provide a bridge to also obtain the output of the analysis of the ABGW tool.    
    
    
Located at `tools/abgw_bridge`, it receives a description    
of an ABE scheme written in JSON, for instance, for the YJR+13 [5] scheme:    
    
```json    
{    
    "scheme_id": "yjr13",    
    "analysis": "abgw",    
    "k": ["a * (1 / x1) + x2 * b + r * (b / bp)", "r * bp * (1 / x1)", "r * b"],    
    "c": ["s", "s / bp"],    
    "mpk": ["bp"],    
    "gp": ["b"],    
    "key" : "a * s",    
    "unknown_vars" :  ["a", "r", "s", "b", "bp"],    
    "known_vars" : ["x1", "x2"]    
}    
```    

It can be used to obtain the corresponding input for ABGW, for instance:
    
```bash
$ python abgw_bridge_cmd.py example.json 
[*] ABGW bridge cmd tool
[!] Processing example.json

params c1,c2,c3,c4,c5,c6 in Zp.
vars a,r,s,b,bp in Zp.
params x1,x2 in Zp.


c1*(a*s/x1 + b*s*x2 + b*r*s/bp) +
c2*(a*s/(bp*x1) + b*s*x2/bp + b*r*s/bp*bp) +
c3*(bp*r*s/x1) +
c4*(r*s/x1) +
c5*(b*r*s) +
c6*(b*r*s/bp)
= a * s.

go.
```

#### ABGW docker tool

It invokes the ABGW ggm analyzer proposed by [ABGW17] in a docker container and
analyzes the inputs provided in `solver_inputs`.

Located at `tools/abgw_docker`, it invokes the ABGW tool with ABE schemes defined in the `solver_inputs` directory.
Note that the `build_and_run.sh` script compiles the image defined in the docker file and launch a container image where ABGW is executed.

For instance:

```bash
$ ./build_and_run.sh               
Sending build context to Docker daemon  48.13kB
Step 1/27 : FROM ubuntu:16.04
 ---> b6f507652425
Step 2/27 : RUN apt update -y
 ---> Using cache
 ---> 1688783f3dcc
Step 3/27 : RUN ln -fs /usr/share/zoneinfo/America/New_York /etc/localtime
 ---> Using cache
 ---> 0dbe43f37870
Step 4/27 : RUN apt-get install -y tzdata
 ---> Using cache
 ---> e3f3790737f4
Step 5/27 : RUN dpkg-reconfigure --frontend noninteractive tzdata
 ---> Using cache
 ---> e6d34b8176ad
Step 6/27 : RUN apt-get --assume-yes install software-properties-common
 ---> Using cache
 ---> 9ed00e1180b3
Step 7/27 : RUN echo "export GGM_PATH=/root/ggm-symbolic-solver" >> /etc/bash.bashrc
 ---> Using cache
 ---> c09a02d4d6f1
Step 8/27 : RUN apt install git vim build-essential sudo python3-dev wget flex bison python3-pip libssl-dev libgmp10 libgmp-dev git openssl -y
 ---> Using cache
 ---> 0107f2df09fb
Step 9/27 : RUN apt-get install -y curl ocaml ocaml-native-compilers opam libtool libtool-bin libgmp-dev libffi-dev m4 libz-dev libssl-dev camlp4-extra
 ---> Using cache
 ---> 629689e6d19b
Step 10/27 : WORKDIR /root
 ---> Using cache
 ---> d79f0bee8e6f
Step 11/27 : RUN git clone https://github.com/miguel-ambrona/ggm-symbolic-solver
 ---> Using cache
 ---> 1a87191dcd28
Step 12/27 : WORKDIR /root/ggm-symbolic-solver
 ---> Using cache
 ---> d9117f162a43
Step 13/27 : RUN opam init --yes
 ---> Using cache
 ---> ea83d8958f44
Step 14/27 : RUN eval `opam config env`
 ---> Using cache
 ---> 3744bac4bc53
Step 15/27 : RUN opam pin add symbolic-solver . -n --yes
 ---> Using cache
 ---> 57ee5bb1a57a
Step 16/27 : RUN opam install symbolic-solver --deps-only --yes
 ---> Using cache
 ---> d151dd1a053f
Step 17/27 : RUN export GGM_PATH=/root/ggm-symbolic-solver/
 ---> Using cache
 ---> fc1963909dd5
Step 18/27 : RUN apt-add-repository -y ppa:aims/sagemath
 ---> Using cache
 ---> d74a3b71a915
Step 19/27 : RUN apt-get update -y
 ---> Using cache
 ---> e66d9de8839d
Step 20/27 : RUN apt-get --assume-yes install sagemath-upstream-binary
 ---> Using cache
 ---> 0dc912223b53
Step 21/27 : COPY ggm_setup.sh/ .
 ---> Using cache
 ---> 54581c89d3eb
Step 22/27 : RUN chmod +x ggm_setup.sh
 ---> Using cache
 ---> 99be81b5d1b0
Step 23/27 : RUN ./ggm_setup.sh
 ---> Using cache
 ---> 2caff98e273d
Step 24/27 : RUN rm -rf examples/*
 ---> Using cache
 ---> ff8f9b8c477b
Step 25/27 : COPY solver_inputs/* examples/
 ---> Using cache
 ---> 4df8886f035e
Step 26/27 : COPY changes/* .
 ---> Using cache
 ---> c62e8b35a2c7
Step 27/27 : CMD ["/bin/bash"]
 ---> Using cache
 ---> eb18b1849e3d
Successfully built eb18b1849e3d
Successfully tagged abeattacks:latest


[*] Now run run_examples.py

root@5f232cb06e7c:~/ggm-symbolic-solver# 
root@5f232cb06e7c:~/ggm-symbolic-solver# ./run_examples.py 

Initialized solver!

./examples/cp_abe_ndcw15.ggm  Complete output:
[...]
```   

## References

- [1] https://eprint.iacr.org/2020/460
- [2] https://eprint.iacr.org/2017/233
- [3] https://eprint.iacr.org/2022/1415
- [4] K. Yang, and X. Jia, “Attribute-Based Access Control for Multi-Authority Systems
in Cloud Storage”, in 2012 32nd IEEE International Conference on Distributed
Computing Systems, pp. 536–545, IEEE, 2012.
- [5]  K. Yang, X. Jia, K. Ren, B. Zhang, and R. Xie, “DAC-MACS: Effective Data
Access Control for Multiauthority Cloud Storage Systems”, in TIFS, 8(11), pp.
1790–1801, IEEE, 2013.

