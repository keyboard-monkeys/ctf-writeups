*What a mundane song, it's just the same note repeated over and over. But could there perhaps be two different notes?*

[audio_frequency_stego.wav](https://github.com/keyboard-monkeys/ctf-writeups/blob/main/2021-hsctf/data/audio_frequency_stego.wav)

The audio contains pulses, which can be considered a clock. Between the pulses can be two very slightly different tones. They can be visualised with for example https://academo.org/demos/spectrum-analyzer/.

The different tones can be translated to binary `011001100110110001100001011001110111101101110011011011000011000101100111011010000101111101110000001100010111010001100011011010000101111101100011011010000011010001101110011001110011001101111101`, which is the flag `flag{sl1gh_p1tch_ch4ng3}`.