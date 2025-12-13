# MCU Heap Fragmentation test

MCUでメモリを動的確保したときの動作を確認するテストコード。

## Environment

- target : NUCLEO-H563
- IDE : STM32CubeIDE v2.0.0
- build tool : GNU Tools for STM32 (13.3.rel1)
    - Runtime library : Reduced C ( `--specs=nano.specs` )

## Tests

下記の手順でメモリを確保し、確保されたメモリの先頭アドレスを確認する。

1. 1024KBのメモリを10個malloc()で8個動的確保する
2. 確保したうち中間の4つ（[1], [3], [5], [6]）をfree()する
3. 512byte, 1024byte, 2048byteのメモリを動的確保する
4. 3をfree()して、2048byte, 1024byte, 512byteのメモリを動的確保する

コードは `Core/Src/main.c` の `run_fragmentation_test()` を参照。

## Result

```
===== Running fragmentation test =====
  unit size = 1024, array size = 8

malloc() memory[]
  memory[0] address = 0x200006c0
  memory[1] address = 0x20000ac8
  memory[2] address = 0x20000ed0
  memory[3] address = 0x200012d8
  memory[4] address = 0x200016e0
  memory[5] address = 0x20001ae8
  memory[6] address = 0x20001ef0
  memory[7] address = 0x200022f8

free() memory[1], [3], [5], [6]

malloc() halfUnitSize -> unitSize -> uintSizeX2
  halfUnitSize = 0x20000ac8
  unitSize     = 0x200012d8
  uintSizeX2   = 0x20001ae8

free() halfUnitSize, unitSize, uintSizeX2

malloc() uintSizeX2 -> unitSize -> halfUnitSize
  halfUnitSize = 0x200012d8
  unitSize     = 0x20000ac8
  uintSizeX2   = 0x20001ae8

free() ALL
```

- free()した領域は再利用されている
- 連続する複数領域をfree()した際、一つの空き領域として扱われている
- 空き領域の探索は、ヒープの空き領域の先頭アドレスから要求サイズを満たす領域を割り当てている（？）
