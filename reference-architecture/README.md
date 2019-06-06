# Reference Architecture
## Introduction
## Architecture overview
```
                  +----------------------------------------+
                  |                                        | [External-Entities]
                  | OSS/BSS/EMS    DHCP    AAA     Policy  |
                  |                                        +------------------------+
                  |                                        |                        |
                  +--------------------+-------------------+                        |
                                       |                                            |
                                       |                                            |
                                       | NETCONF                                    | NETCONF
                                       |                                            |
                                       |                                            |
                    +------------------+----------------------+                     |
                    |                                         |                     |
                    |               [vBNG-CP]                 |                     |
                    |                                         |                     |
                    +----+---------------+---------------+----+                     |
                         |               |               |                          |
 + - - - - - - - - - - - | - - - - - - - | - - - - - - - | - - - - - - - - +        |
   [Service-Interface]   |     [Control-Interface]  [Management-Interface]          |
 |        VXLAN          |            S-CUSP             |     NETCONF     | [vBNG] |
                         |               |               |                          |
 + - - - - - - - - - - - + - - - - - - - | - - - - - - - | - - - - - - - - +        |
                         |               |               |                          |
                    +----+---------------+---------------+-+                        |
                    |                                      |                        |
                    |              [vBNG-UP]               +------------------------+
                    |                                      |
                    +-------------------+------------------+
                                        |
                                        |
                               +--------+-----------+
                               |                    |
                               |  [Access-Network]  |
                               |      (SEBA)        |
                               +--------+-----------+
                                        |
                                   +----+----+
                                   |   User  |
                                   +---------+
```
## Access Network
- BBF: [TR-383](https://www.broadband-forum.org/download/TR-383.pdf)
- Implementation: [SEBA](https://www.opennetworking.org/seba/)

## CU-Separation
Following components are based on the draft
[draft-cuspdt-rtgwg-cu-separation-yang-model-03](https://tools.ietf.org/html/draft-cuspdt-rtgwg-cu-separation-yang-model-03).
In the draft are defined three yang models:

1. Yang model for the Management Interfaces (ietf-vbng)
2. Yang model for the Control Plane configuration (ietf-vbng-cp)
3. Yang model for the User Plane configuration (ietf-vbng-up)

### vBNG
#### Service Interface
- IETF: [VXLAN GPE extension](https://tools.ietf.org/html/draft-hu-nvo3-vxlan-gpe-extension-for-vbng-01)

#### Control Interface
- IETF: [CUSP](https://tools.ietf.org/html/draft-cuspdt-rtgwg-cu-separation-bng-protocol-03)

#### Management Interface
- IETF: [Management Interfaces](https://tools.ietf.org/html/draft-cuspdt-rtgwg-cu-separation-yang-model-03#section-3.1)
- YANG: [ietf-vbng](https://github.com/YangModels/yang/blob/master/experimental/ietf-extracted-YANG-modules/ietf-vbng@2019-03-08.yang)
- augment: [LNE](https://tools.ietf.org/html/draft-ietf-rtgwg-lne-model-10)


### vBNG-CP
- IETF: [vBNG-UP](https://tools.ietf.org/html/draft-cuspdt-rtgwg-cu-separation-yang-model-03#section-3.3)
- YANG: [ietf-vbng-up](https://github.com/YangModels/yang/blob/master/experimental/ietf-extracted-YANG-modules/ietf-vbng-up%402019-03-08.yang)
- augment: [LNE](https://tools.ietf.org/html/draft-ietf-rtgwg-lne-model-10)


### vBNG-UP
- IETF: [vBNG-CP](https://tools.ietf.org/html/draft-cuspdt-rtgwg-cu-separation-yang-model-03#section-3.2)
- YANG: [ietf-vbng-cp](https://github.com/YangModels/yang/blob/master/experimental/ietf-extracted-YANG-modules/ietf-vbng-cp%402019-03-08.yang)
- augment: [LNE](https://tools.ietf.org/html/draft-ietf-rtgwg-lne-model-10)


### External Entities
- BBF: [TR-384](https://www.broadband-forum.org/download/TR-384.pdf)
