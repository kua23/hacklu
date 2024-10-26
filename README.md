# hacklu

## Pump it up
`import secrets
from Crypto.Cipher import AES # pip install pycryptodome
import os
import string
import audio_engine`

`flag = flag{y0u_
with open("../SECRET/flag.txt", "r") as fp:
    flag = fp.read().strip()
flag = flag.encode().hex()`

`if not os.path.isdir("snippets"):
    os.mkdir("snippets")`

` for cnt,s in enumerate(string.digits + "abcdef"):
    os.system(f"ffmpeg -ss {cnt} -t 1 -i pump.opus snippets/{ord(s)}.voc")
sound_bites = []
for s in flag:
    out = audio_engine.extract_sound_data(f"snippets/{ord(s)}.voc")
    sound_bites.append(out)
audio_engine.create_voc_file("flag.voc", b"".join(sound_bites)) 
`

`def encrypt_it():
    key = secrets.token_bytes(16)
    cipher = AES.new(key, AES.MODE_ECB)
    with open("flag.voc", "rb") as f:
        val = f.read()
    val += b"\x00" * (16 - len(val) % 16)
    with open("flag.enc", "wb") as f:
        f.write(cipher.encrypt(val))
encrypt_it()`


Here we try to recover the plaintext flag by exploiting the pattern of repeated AES-ECB blocks in the flag.enc file.


The original script reads the flag from a file, converts it to a hexadecimal and uses this hex flag string to generate short audio snippets for each character.
For each hex character, the script extracts 1-second audio segments from pump.opus and saves these as separate files in the snippets folder.
All these snippets are then combined into a single flag.voc file.

The flag.voc file is padded to a 16-byte boundary, then encrypted using AES in ECB mode. The resulting ciphertext is saved in flag.enc.

It encrypts identical plaintext blocks to identical ciphertext blocks, which leaks information about repeating patterns in the plaintext.

The solution script opens flag.enc and reads it in 16-byte blocks, skipping between two predefined offsets (OFFSET_ONE and OFFSET_TWO).

Each 16-byte block is stored in a list datasets, and an ordered list of unique blocks is created, keeping the blocks’ order of appearance intact.

Unique blocks in datasets are assigned placeholder characters (from G to Z), which are stored in out. This string now represents the encrypted flag in terms of placeholder characters for each block.

Since the flag format is kinda known (flag{y0u_ at the start and } at the end), the code replaces placeholder characters with their known hex values.
Using mappings, it maps placeholders for known segments of the flag and gradually replaces these placeholders in out with actual values.

Once mapped, the placeholders in out represent a mix of known and unknown hex values. The solution attempts to convert this reconstructed hex string into ASCII characters to guess the plaintext flag.

The final output displays the partially reconstructed flag in ASCII format, indicating where gaps remain or where placeholders are still unreplaced.
This approach leverages the repetition of blocks due to ECB mode to deduce parts of the flag based on known patterns and placeholder replacements. By partially knowing the format, it approximates the actual flag without decrypting it directly.
