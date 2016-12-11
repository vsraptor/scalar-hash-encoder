###SDR : Scalar Matrix-Hash encoder

#### What is this ?

When you work with *Sparse Distributed Representation* /SDR/ data, you need to convert the integer datapoints
into SDR format /which in a nutshell is very sparse binary string/ : (http://ifni.co/bbHTM.html).

So the library in this repo is one such Encoder that converts **Integer =to=> SDR**.
The peculiarity of this specific encoder is that it uses Matrix based hash functions.

#### How it works ?

The goal as I already said is to generate a sparse binary string (SDR).
The accepted standard is to have sparsity of 2%, which would mean f.e. that if the generated SDR is 
2048 bits long we would have 40-41 bits ON i.e. 2048 * 0.02.

To do this we create a randomly populated 0|1 - matrix of size NxM.
To figure N and M we have to know what are the maximum input value and the number of output bits (2048 in our case).

Let say the maximum input value is 20_000.
That means we would need *log2(2048) = 10* rows and *log2(20_000) =~ 15* cols.
After that the calculation is very simple.

	1. We do AND operation of every row of the matrix with the input value.
	2. We count all 1-bits for every row of the result
	3. If the count of 1s is ODD this means the result bit will be 1, if the count is EVEN the result bit will be 0
	4. Convert this **binary string** to **Integer** /idx/.

So this *Integer* is the index of bit in the output SDR that has to be set to 1.
Again for our specific case we have 40 such random matricies to generate 40 bits.

That is it. Here is how sample hash-matrix looks like.


```python

 0| 110101111000011
 1| 100000001100010
 2| 001110011011000
 3| 111001010001110
 4| 000110001001000
 5| 110001110011101
 6| 010110000000100
 7| 001110100010010
 8| 100000011110000
 9| 101100001000110
10| 010000111100000

```



#### Example usage

```python

:from encoders.scalar_mxhash_encoder import *

: se = ScalarMxHash(15000, nbits=128, sparsity=0.1)

: se.encode(5)
: bitarray('00000000001000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000001000000000000000000000000')

: se.encode(55)
: bitarray('00000000000000000000000000000000000000000000000000001000000000000000000000000000000000100000000000000000000000000000000000000000')

: d = [ se.encode(np.random.randint(0,15000)) for x in range(50) ]
: se.collision_cnt
: 16
: se.avg_collision
: 1.9865038044884135e-12



: se = ScalarMxHash(max_in=20000, nbits=2048, sparsity=0.02)
: d = [ se.encode(np.random.randint(0,15000)) for x in range(1000) ]
: se.avg_collision
: 1.0110804927657982e-15
: se.collision_cnt
: 337

```