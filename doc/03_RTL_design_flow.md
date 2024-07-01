The RTL design flow should follow the steps:

1. High-level functional modeling
In this step, the designer models the HW component that needs to be implemented. This model needs to correspond 1:1 with the specification, but its correctness is fundamental. The model implementation should be straightforward and based on high-level languages like Pyhton, Matlab, C.
Below is an example with an adder:

```python
def nbit_signed_int_adder_model(addend1 : int, addend2: int, bitwidth: int, ovf : bool = True): -> int
	if (addend1>2**(bitwidth-1)-1 or addend1<-2**(bitwidth-1)): raise(ValueError)
	if (addend2>2**(bitwidth-1)-1 or addend2<-2**(bitwidth-1)): raise(ValueError)
  result = addend1 + addend2
	#example on overflow model
	if ovf:
		if(result>2**(bitwidth-1)): result= result - 2**(bitwidth)
	return result

def nbit_signed_int_adder_ovf_check(addend1 : int, addend2: int, bitwidth: int): -> bool
	result = addend1 + addend2
	if (result>2**(bitwidth-1)-1 or result<-2**(bitwidth-1)): return True
	return False
```

2. Testbench and test design
If you design your test environment beforehand, the design phase will be faster and smoother. You can always iterate and improve this during the project.
3. HDL design
If everything is in place, the design should be smooth and you should be able to test every component and every level of your design with ease and fast.
