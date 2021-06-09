# ECOND_ROC_Emulator
Emulating HGCROC for ECOND 

### Emulating datasets
- N: number of orbits
- sequence: string with sequence of L1as e.g.:
```
python3 simulateInputECOND.py -N 10692 --bcr --sequence random 
python3 simulateInputECOND.py -N 10692 --bcr --sequence fixed --L1a_freq 1 --nL1a 10
```
- to send non-zero data:
```
python3 simulateInputECOND.py -N 10692 --bcr --sequence fixed --L1a_freq 50 --nL1a 3 --format-data
```
note that `fixed` needs L1a_freq and nL1a

- ecr: event counter reset, e.g.:
```
python3 simulateInputECOND.py -N 10692 --bcr --sequence random --ecr --ecrBX 9050 
```
- ebr: event buffer reset, e.g.
  - 10692 fc with 2 L1as sent at a fixed frequency (50 BX) and a EBR sent after those events finish transmitting to ECON
  
  ```python3 simulateInputECOND.py -N 10692 --bcr --sequence fixed --L1a_freq 50 --nL1a 1 --ebr --ebrBX 150```
  - 10692 fc with 3 L1as sent at a fixed frequency (50 BX) and a EBR sent in between those events 
  
  ```python3 simulateInputECOND.py -N 10692 --bcr --sequence fixed --L1a_freq 50 --nL1a 2 --ebr --ebrBX 98```
  
  - 10692 fc with 3 L1as sent at a fixed frequency (50 BX) and a EBR sent after the 2nd L1a is sent but before the ROC has begun transmitting that event. Then I send another L1A later.
  
  ```python3 simulateInputECOND.py -N 10692 --bcr --sequence fixed --L1A_freq 50 --nL1a 2 --ebr --ebrBX 102```
  
  - 10692 fc with 3 L1as sent at a fixed frequency (50 BX) and a EBR sent after the 2nd L1a is sent and after the ROC has begun transmitting that event (the ROC is in the middle of reading that event - has only read word 1). Then I send another L1a later.
  
  ```python3 simulateInputECOND.py -N 10692 --bcr --sequence fixed --L1a_freq 50 --nL1a 2 --ebr --ebrBX 108```


#### Formated dataset
We have 32 bits, broken down into two sets of 16.  We can further break down the 16 into a set of 4 bits for counting, a set of 4 bits for eLink Number and a set of 8 bits for packet word number.

The packet word number would be as follows:
```
Header -              0
CM     -              1
Ch 0   -              2
Ch 1   -              3
Etc up to CRC
````
 
The eLink Number would be as follows:
```
eLink 0  -              1
eLink 1  -              2
eLink 2  -              3
Etc up to eLink 11
```

The count would simply be a packet count from 0 to 15 and then roll over. 

We would then simply repeat this 16 bit pattern twice to make the 32 bit pattern.
These patterns have no meaning as far as the eLink processors are concerned.
Ultimately the 32 bit word would be:

```
<4-bit Count><4-Bit eLink #><8-bit Packet Word #><4-bit Count><4-Bit eLink #><8-bit Packet Word #>. 
```

The first packet from eLink 0 would look as follows:
```
1.       Header (standard header word)
2.       CM                 -              0000 0001 00000001  0000 0001 00000001
3.       Ch 0               -              0000 0001 00000010  0000 0001 00000010
4.       Ch 1               -              0000 0001 00000011  0000 0001 00000011
5.       Ch 2               -              0000 0001 00000100  0000 0001 00000100
6.       Etc.
```

The second packet from eLink 1 would look as follows:
```
1.       Header (standard header word)
2.       CM                 -              0001 0010 00000001  0001 0010 00000001
3.       Ch 0               -              0001 0010 00000010  0001 0010 00000010
4.       Ch 1               -              0001 0010 00000011  0001 0010 00000011
5.       Ch 2               -              0001 0010 00000100  0001 0010 00000100
6.       Etc.~
```