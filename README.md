# Encryption Challenge

Create a program that, when given a message from a file and an integer key, encrypts the message by adding the ASCII number for each character to the character's index and to the key and then converts that sum to binary. The program should also be able to decrypt any encrypted messages it creates. Make the program as fast as possible in the language of your choice.

# My Solution

I chose to write my solution in C to make it as fast as (reasonably) possible. At first, I stored encrypted messages in text files containing ASCII 1s and 0s, but I moved from that to use literal binary files which sped up the program tremendously and made the algorithm much simpler.

## Encryption

Integers are stored as 32-bit binary numbers. So, using binary files saved me the trouble of converting each calculated number to a 32-bit binary number. To encrypt files, I read the file character-by-character:

```c
char buf[1]; // buf is each character
while (!feof(infile)) {
    fread(buf, sizeof(buf), 1, infile);
    ...
}
```

Then I calculate the "ASCII code" from ASCII number plus the key and the index which will be converted to binary when written:

```c
int ascii_code = (int)*buf + KEY + i;
fwrite(&ascii_code, sizeof(ascii_code), 1 , outfile);
```

## Decryption

Again, using binary files saved me a lot of trouble when decrypting. To decrypt files, I made my buffer an integer which is a 32-bit binary number, so the program will read every 32 bits into the buffer from the binary file: 

```c
int buf;
while (!feof(infile)) {
    fread(&buf, sizeof(buf), 1, infile);
    ...
}
```

Then I do the reverse of encryption by subtracting each integer by the key and its own index:

```c
buf -= KEY + i;
```

## Output

The program accepts a file as the input using the `-i` flag and outputs to a file using the `-o` flag. Both of these options are required. I added a `-v` flag to print a verbose output of what the program was writing to the output file. This meant I had to use part of my old ASCII-based algorithm to print to the console during encryption. This algorithm iterates in reverse over a 32-length integer array:

```c
int binary_num[32];
for (int j = 31; j >= 0; j--) {
    ...
}
```

If the `ascii_code` does not equal to 0, it will mod the it by 2. When dividing a number by 2 there are only two possible remainders it can return, 1 or 0. Then it will set `ascii_code` to itself divided by 2:

```c
if (ascii_code != 0) {
    binary_num[j] = ascii_code % 2;
    ascii_code /= 2;
}
```

So if the `ascii_code` was 12, the program will convert it to binary as follows:

```
12 mod 2 = 0    12/2 = 6
 6 mod 2 = 0    6/2 = 3
 3 mod 2 = 1    3/2 = 1
 1 mod 2 = 1    1/2 = 0

Result: 1100
```

If the `ascii_code` equals 0, meaning the entire number has been converted to binary, it will fill the remaining indices in the `binary_num` array with zeroes to ensure all 32 indices have a value so it accurately represents the 32-bit integer. Then the program iterates over the `binary_num` array, printing each index:

```c
for (int j = 0; j < 32; j++) {
    printf("%d", binary_num[j]);
}
```

When printing during decryption, the program simply prints the ASCII character that the buffer integer corresponds to:

```c
printf("%c", (char)buf);
```

# Usage

## Encryption

To encrypt, specify your input and output files. Then use the `-e` flag with your key:

```
./bincrypt -e <key> -i <input_file> -o <output_file>
```

For a verbose output during encryption, use the `-v` flag:

```
./bincrypt -e <key> -i <input_file> -o <output_file> -v
```

## Decryption

To decrypt, again, specify your input and output files. Then use the `-d` flag with your key:

```
./bincrypt -d <key> -i <input_file> -o <output_file>
```

For a verbose output during encryption, use the `-v` flag:

```
./bincrypt -d <key> -i <input_file> -o <output_file> -v
```
