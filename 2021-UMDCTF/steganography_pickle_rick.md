We were given two sound files. https://drive.google.com/drive/folders/135epDZ18MdIycbBt_Fekbi8hdF-83v6Y

Using the command `steghide extract -sf together-forever-encoded.wav` without a passphrase, we can extract the file `steganopayload318205.txt`, which contains the text `The password is "big_chungus"!`.

Now using the command `steghide extract -sf rickroll.wav` with the passphrase `big_chungus`, we can extract the file `steganopayload318287.txt`, which contains the flag `UMDCTF-{n3v3r_g0nna_l3t_y0u_d0wn}`.