#!/usr/bin/python3
"""
frequence bin script

"""

NSLICE = 10
FREQ_INTERVALS = [(50, 500)] + [(x * 500 + 1, (x + 1) * 500) for x in range(1, 40)]

__version__ = "4"
__version_date__ = "2020-03-27"

import argparse
import math
import os
import pathlib
import sys

import parselmouth
from parselmouth.praat import call

parser = argparse.ArgumentParser(prog="lin_freq_bin",
                                 usage="\nlin_freq_bin -d sounds_directory_path -o OUTPUT.tsv",
                                 description="Linear frequency bin with Praat")
parser.add_argument("-d", "--directory", required=True, action="store", dest="directory", help="Set directory of WAVE files")
#parser.add_argument("-c", "--cpu", action="store", dest="cpu", default=1, type=int, help="Set number of CPU/cores to use (default 1)")
parser.add_argument("-o", "--output", action="store", dest="output", help="Set path for the output file")
parser.add_argument("-v", "--version", action='version', version=f"%(prog)s v.{__version__} {__version_date__}")
parser.add_argument("-i", "--initial-silence", action='store', dest="initial_silence", default=0, type=float,
                    help="indicate the silence duration (in s) at the beginning of the sound")
parser.add_argument("-f", "--final-silence", action='store', dest="final_silence", default=0, type=float,
                    help="indicate the silence duration (in s) at the end of the sound")

args = parser.parse_args()

if not args.directory:
    print('Sounds directory not found!')
    sys.exit(3)

if not os.path.isdir(args.directory):
    print(f"{args.directory} is not a directory")
    sys.exit(1)

print(f"Sound files directory path: {args.directory}")

# header
header = ["file name", "duration"]
for slice_ in range(1, NSLICE + 1):
    for idx, freq_int in enumerate(FREQ_INTERVALS):
        header.append(f"mb{idx + 1}_sl{slice_}_dB")
out = "\t".join(header) + "\n"

files_list = sorted(list(pathlib.Path(args.directory).glob("*.wav")) + list(pathlib.Path(args.directory).glob("*.WAV")))
if not files_list:
    print(f"No files found (.wav or .WAV)")
    sys.exit(1)


for file_name in files_list:

    if args.initial_silence or args.final_silence:
        dummy_sound = parselmouth.Sound(str(file_name)) 
        try:
            sound = dummy_sound.extract_part(args.initial_silence,
                                             dummy_sound.duration - args.final_silence,
                                             parselmouth.WindowShape.RECTANGULAR, 1, False)

        except Exception:
            print((f"Error on {file_name.stem} parselmouth.PraatError: Extracted Sound would contain no samples. "
                   "Sound: part not extracted."),
                   file=sys.stderr)
            continue
    else:
        try:
            sound = parselmouth.Sound(str(file_name))
        except parselmouth.PraatError:
            print((f"Error on {file_name.stem} parselmouth.PraatError: Extracted Sound would contain no samples. "
                   "Sound: part not extracted."),
                   file=sys.stderr)
            continue
            

    sound.scale_peak(0.99)

    energy_values = [file_name.stem, round(sound.duration, 3)]

    ws = sound.duration / NSLICE

    for slice_ in range(1, NSLICE + 1):
        bp = (slice_ -1) * ws
        fp = slice_ * ws

        sound_slice = sound.extract_part(bp, fp, parselmouth.WindowShape.RECTANGULAR, 1, False)

        sound_slice_spectrum = sound_slice.to_spectrum()

        for freq_int in FREQ_INTERVALS:
            try:
                energy = 20 * math.log10(sound_slice_spectrum.get_band_energy(freq_int[0],  freq_int[1])/0.00002)
                energy_values.append(round(energy, 2))
            except:
                print((f"Error on {file_name.stem}, {slice_}, {freq_int}, "
                       f"{sound_slice_spectrum.get_band_energy(freq_int[0],  freq_int[1])}"),
                      file=sys.stderr)
                energy_values.append("NA")

    out += "\t".join([str(x) for x in energy_values]) + "\n"

if args.output:
    try:
        with open(args.output, "w") as f_out:
            f_out.write(out)
    except Exception:
        print(f"\nError saving results in {args.output}.\nFile {args.directory}.tsv created.", file=sys.stderr)
        with open(f"{args.directory}.tsv", "w") as f_out:
            f_out.write(out)
else:
    print(out)
