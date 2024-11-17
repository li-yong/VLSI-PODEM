# VLSI-Testing Project 4

This document explains the project 4 BIST implementation. 

# Logical Flow
```  
Load and Initialize the circuit.
Init TPG_LFSR
Init ORA_MISR

Loop From 1 to 100:
	Break Loop if all SA faults were detected

	TPG Generate a 32bit test pattern.  e.g 10110101010101010101011101001010
	Simulate good circuit Output.  e.g  `Good circuit output: 10`
	MISR calculate the golden_signature. e.g. `Golden signature calculated: 0100000000000000`

	For each SA errors in circuit:
		Reset circuit to error free
		inject the SA
		Simulate circuit output
		MISR calculate the circuit output signature. 

		IF (Sig != Sig_golden):
			Error_Detected; remove SA from circuit; 
			continue next SA

		IF (OP == OP_golden) and (Sig == Sig_golden):
			Error_Not_Detected
			continue next SA
		
		IF (OP != OP_golden) and (Sig == Sig_golden):
			aliasing += 1

	
Print BIST Result
```			



# Usage and Demo
## Invoke

Invoke with `-action ATPG` with `-file_isc`. Then program will read the circuit definition file then proceed to ATPG function which are depicted as following.

```
$ ./cpp_isc_parser.exe -action BIST -file_isc ../ISCAS-85/c17.isc 
```

## Initiate Circuit and TPG_LFSR
```
ISC file: ../ISCAS-85/c17.isc

Circuit Gates:
Num of Gate 17
Num of PI 5
Num of PO 2
Circuit initialized.

Initialize TPG_LFSR with 10101010101010101010101010010101

```

## TPG and ORA
1. TPG_LFSR runs one clock to generate a test pattern.
1. Simulate circuit to get the good circuit output.
1. ORA_MISR generate the signature based on the circuit output.
```
Loop 0, TPG generated test pattern 10110101010101010101011101001010
Good circuit output was simulated. Good circuit output: 10
Calculating signature on ORA. Golden signature calculated: 0100000000000000
```

## Inject SA Fault 
Following show two scenarios, the first one didn't detect the SA while the second detected the SA.
```
== Injecting SA1 on 1gat. Remaining sa error: 22
po match, po_output_string 10 , golden_output_string 10
        Golden         signature: 0100000000000000
        Faulty circuit signature: 0100000000000000


== Injecting SA0 on 3gat. Remaining sa error: 22
po does not match, po_output_string 00 , golden_output_string 10
        Golden         signature: 0100000000000000
        Faulty circuit signature: 0000000000000000
signature of fault circuit does not match golden signature.
stuck error detected, removed SA_ERROR >sa0@3gat from the SAlist.

```
## Show Summary

```
=== BIST Simulation Completed ===
Circuit              : ../ISCAS-85/c17.isc
Total SA Errors      : 22
Detected Errors      : 22
Detection Ratio      : 1
Alias Cnt            : 0
Loop  Cnt            : 3
```


## Test
### LFSR Test
`test_case_111`, verify LFSR function. Polynomial `H(x)= X^3+x+1`. LFSR initial state `111`.

```
./cpp_isc_parser.exe  -action test_case_111 -file_isc ../ISCAS-85/c17.isc

initial 111
101
100
010
001
110
011
111
101
```

### MISR Test
`test_case_01010001`, verify MISR function. Polynomial `H(x)=X^5+X^3+X+1`, input stream `01010001`.

```
./cpp_isc_parser.exe  -action test_case_01010001 -file_isc ../ISCAS-85/c17.isc
	misr_loop 0, input 1, output: 10000
	misr_loop 1, input 0, output: 01000
	misr_loop 2, input 0, output: 00100
	misr_loop 3, input 0, output: 00010
	misr_loop 4, input 1, output: 10001
	misr_loop 5, input 0, output: 10010
	misr_loop 6, input 1, output: 11001
	misr_loop 7, input 0, output: 10110

```

