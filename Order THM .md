
Cypher ----


We intercepted one of Cipher's messages containing their next target. They encrypted their message using a repeating-key XOR cipher. However, they made a critical error—every message always starts with the header:

ORDER:
Can you help void decrypt the message and determine their next target?
Here is the message we intercepted:

1c1c01041963730f31352a3a386e24356b3d32392b6f6b0d323c22243f6373

1a0d0c302d3b2b1a292a3a38282c2f222d2a112d282c31202d2d2e24352e60


Solution ----

first convert order to hex 

ORDER: -> 4f 52 44 45 52 3a

next take first six bytes same as headers bytes and calculate xor with the other header use xor calculator webpage

1c1c01041963730f31352a3a386e24356b3d32392b6f6b0d323c22243f63731a0d0c302d3b2b1a292a3a38282c2f222d2a112d282c31202d2d2e24352e60

So when calcultaed xor 4f524445523a + 1c1c01041963 --> 534e45414b59

then calculate xor wiht both by expanding them same as length of cipher

1c1c01041963730f31352a3a386e24356b3d32392b6f6b0d323c22243f63731a0d0c302d3b2b1a292a3a38282c2f222d2a112d282c31202d2d2e24352e60
                                                        +
534e45414b59534e45414b59534e45414b59534e45414b59534e45414b59534e45414b59534e45414b59534e45414b59534e45414b59534e45414b59534e
                                                        =
4f524445523a2041747461636b206174206461776e2e205461726765743a2054484d7b7468655f6861636b66696e6974795f686967687363686f6f6c7d2e

now decode hex in cyberchef to get this 

ORDER: Attack at dawn. Target: THM{the_hackfinity_highschool}.