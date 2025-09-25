# Solving the Sebucsert Cryptosystem

by Eclipixie, loki314  

Flag: `pecan{clarity_through_cubes}`

We were the first team to solve the Sebucsert Cryptosystem challenge, two years after it went live in Pecan+ 2023 as that year’s ASD special, a CTF for high school students in Years 10-12 run by various professionals across the country. Notably, the challenge had remained unsolved by the organisers.

## Starting out

Here is the information we had to start with:

* We also know that the flag will be formatted as `pecan{flag}`

### Excerpts from [message.txt](../resources/sebucsert/message.txt)

```
WSZHLCC

ANS_YZRIVCSU_KZBLRZRSYZRKZBXKKPNKZKVCQZIND_CNWDC_RSPCCUPVET_SVZN_RIDSRIBWR_ESYZRKOIQRSNXRVJSDZZIND_DZRSYOCQPOH___UOH_RSYZWRYGECYZRIVCSU_SVZPVDANMOB_CUKXACIND_CPIDSDPOZAVDCEKOG_CKVRZATRKVRZA_KOAQZIOH__YZRSPCCUPND_CYZRKOIQRI_SSVZNZZIVDSZIBWRP_UPOCCSYOCQPOH__YZRSPCCUPND_ABRRJZREA_UPOCCKVCQZIZZABRRJZRECNWDC_RSYOCQPZHST_YZWRYGECYZRIVCSU_KZBLRZRSYZRKZBXKKPNKZSVZPVDANMOB_CUKXACKPB_ZKNIOZRJZ

...

_JZRSNWCFSTPBTCYZRI_OCESU

_JOCSI_SSYZRSNWCFSTPBTCYZRI_OCESU
ZJZRSNWCFSTPBTCYZRI_OCESU_IBTATFK
_JOCSI_SSYZRSNWCFSTPBTCYZRI_OCESU
XIMUKPDQPVGNZSVZPTFKZS_HQPZTU_YZRI_OCESU_IBTATFKZIND_DTZUPTFKZKOCHCGSZJZDPZTPBWCQ_YZHS_NXAMUIPMPAUSYZRI_ODQZECIBTAPMUSYZRI_OXPT
ZXCFZREDZODKZKOKUQ_SVZ_YZRI_OCESU_KOKUQ_SVZ_YZRI_OCESU_IBTATFK
TPE_NXAMUSYZHU__HQPVEYSRECYSVZPTFKZSYODC_OUZEAOCCSYZWSEAOH_
WDD_A_UPVDQNZZSYOCQPTFKZSNXAMUKZWS
GIZTPNDTPVGNZEECYODC_NXAMUKVAMUSYZRSNXDAWZABTATFKZI_FOWCRQ_YZRKOW_ZSIBTCYZRI_OXPT
GIZTPNDTPVGNZEECYZHS_NXAMUSYZRSTZWKCABTATFKZKPCHCGS_YZRK_RWIQCIBTCYZRI_OXPT
BIZTPNDTPVGNZEECNXAMUSYZRIVIUZSIBTCYZRI_OXPTEKZZIVXZ_GSBSDZZICQSEIBTATFK
BEODKZKTZKOKUQ_SVZ_YZRI_OCESU_KOKUQ_SVZ_YZRI_OCESU_IBTATFK
_JOCSI_SSYZRSNWCFSTPBTCYZRI_OCESU
ZJZRSNWCFSTPBTCYZRI_OCESU_IBTATFK
_JOCSI_SSYZRSNWCFSTPBTCYZRI_OCESU

...

ISZID{CPMSWCJT_YZWRYGEC_FEZ}
```

### Excerpts from [sebucsert.pdf](../resources/sebucsert/sebucsert.pdf)

- Loki

“To exchange messages, parties must have pre-shared a 3x3x3 cube comprised of the letters of  
the alphabet, and underscore (‘\_’). Make sure the orientation of the cube is well-established  
with both parties.”  

To summarise, Sebucsert is almost functionally identical to the triphid cipher, with a dynamic period which always equals the length of the line that is being encrypted. This means that we need a decryption key, which has 27\! possibilities (far too many for an exhaustive brute-force). 27\! Is close to 10^28.

There is some reading online about how trifid keys are relative and positions can be defined arbitrarily.

## Bindings and Planes

- Eclipixie

The concepts of ‘bindings’ and ‘planes’ as we called them were quite important to our solution for Sebucsert, so they will be explained here.

Due to the nature of the cube key, it’s not actually required to have the XYZ coordinates of each character. Rather, we only need to know which characters share X, Y or Z coordinates with which other characters.

Intuitively, this leads to a key system made up of three sets of three ‘planes’. Each plane is an unordered set of nine characters, and represents all of the characters holding the same X, Y or Z coordinate. This means that each character must appear in each *set of planes* exactly once. 

```
key_bindings = [` 
  ['q_urtszce', 'ynhkdjgbf', 'wmoavpxil'], # coordinate groups that share x  
  ['yqmuvtjlf', 'worhxzgeb', '_nkadspci'], # coordinate groups that share y  
  ['y_novtpzb', 'wqmhradcf', 'lukgsxjie']  # coordinate groups that share z  
]
```

This set of bindings is an example of a fully discerned key with all relations known, this is the key that was used to encrypt message.txt.

The nature of this system means that partial keys work out of the box, and programs can easily utilise partial keys when trying to decrypt/encrypt/’shift’ text. Additionally, keys do not need to have exactly three planes in each axis, nor do each of those planes need to have exactly 9 characters (they just cannot have more than 9). For instance, we quickly discerned that the string `ISZID{CPMSWCJT_YZWRYGEC_FEZ}` at the very bottom of the encrypted message doc decrypts to `PECAN{!!!!!!!!!!!!!!!!!!!!!}` (with `!` being a placeholder for an unknown character). Knowing this, it’s quite easy to figure out that P, I and A share an X coordinate, as well as S and E, among others. This organises into the groups of `x: PIA SE ZC DN, y: PS ZE IC DA CN, z: PZ IE DCA PM`. Each character in a group shares an X, Y or Z binding with every other character in its group. Groups are separated by spaces, and the axis a group applies to is shown to its left (for example, `x:`)

These were our original bindings, discerned from the relations between the PECAN string and the encrypted ISZID string.

```
initial_bindings = [
  ["es", "cz", "nd", "aip"],
  ["ps", "ez", "ad", "nci"],
  ["ei", "npz", "acd"]
]
```

Compared to the full 27 characters in each coordinate required for a full set of binding this is incapable of providing any useful insight into the plaintext. Using the bindings instead of a full structured key lets us perform decryptions and reveal plaintext where we know there is only one possibility for that character to be.

## Attempts for reliable keygen

- Eclipixie, Loki

After quite a while of trying to ‘play sudoku’ and several failed attempts to find the key, we decided to play the brute-force game: have a computer generate a lot of keys based on our original bindings, and compare the shifted results with English frequency distributions. The idea was that since we could construct partial keys, we would use keys that converged the most on legible English to discern new bindings and repeat until we had a full key.

The main roadblock was figuring out how to generate usable keys based on existing bindings. We tried multiple strategies, including a slightly modified form of wave function collapse, but could never get this function to work consistently. The patchwork fix was to simply recurse if the function failed to generate a valid key, but we quickly found out that the more restrictive our bindings were, the more unreliable the function became. Eventually, we started hitting Python’s recursion limit.

A couple of attempts were made at making this program more reliable by applying additional constraints to the wavefunction collapse criteria, but to no avail.

The main reason for the keygen failing was the constraints that are applied to the format of the cube key that we were using. At the end of generation, the key must have exactly 9 characters in a plane, 3 characters in a row or column, and 1 character in a cell. In terms of bindings, this means that no more than 3 characters may share the same two bindings, and no two characters may share all three bindings.

So for instance, in our key structure: `x: abcd, y: abcd` is an invalid key, since more than three characters share the same row or column. Visually, this is like trying to create a straight line 4 units long in a 3x3x3 cube.

Additionally, `x: ab, y: ab, z: ab` is invalid, since it implies that A and B occupy the same cell, which is impossible.

## Discovery of repeated text variances

- Eclipixie, Loki

We found certain strings that repeat exactly several times throughout the encrypted message, which have different characters preceding/succeeding them. Repeated text fragments are one of the few pieces of information available when discerning a key. If any text patterns of three or more characters exist, a lot of information about the key can be gleaned from analysing the two characters after the repeated pattern.

For instance, an excerpt from the encrypted message.txt reads:
```
_JZRSNWCFSTPBTCYZRI_OCESU

_JOCSI_SSYZRSNWCFSTPBTCYZRI_OCESU
ZJZRSNWCFSTPBTCYZRI_OCESU_IBTATFK
_JOCSI_SSYZRSNWCFSTPBTCYZRI_OCESU
XIMUKPDQPVGNZSVZPTFKZS_HQPZTU_YZRI_OCESU_IBTATFKZIND_DTZUPTFKZKOCHCGSZJZDPZTPBWCQ_YZHS_NXAMUIPMPAUSYZRI_ODQZECIBTAPMUSYZRI_OXPT
ZXCFZREDZODKZKOKUQ_SVZ_YZRI_OCESU_KOKUQ_SVZ_YZRI_OCESU_IBTATFK
TPE_NXAMUSYZHU__HQPVEYSRECYSVZPTFKZSYODC_OUZEAOCCSYZWSEAOH_
WDD_A_UPVDQNZZSYOCQPTFKZSNXAMUKZWS
GIZTPNDTPVGNZEECYODC_NXAMUKVAMUSYZRSNXDAWZABTATFKZI_FOWCRQ_YZRKOW_ZSIBTCYZRI_OXPT
GIZTPNDTPVGNZEECYZHS_NXAMUSYZRSTZWKCABTATFKZKPCHCGS_YZRK_RWIQCIBTCYZRI_OXPT
BIZTPNDTPVGNZEECNXAMUSYZRIVIUZSIBTCYZRI_OXPTEKZZIVXZ_GSBSDZZICQSEIBTATFK
BEODKZKTZKOKUQ_SVZ_YZRI_OCESU_KOKUQ_SVZ_YZRI_OCESU_IBTATFK
_JOCSI_SSYZRSNWCFSTPBTCYZRI_OCESU
ZJZRSNWCFSTPBTCYZRI_OCESU_IBTATFK
_JOCSI_SSYZRSNWCFSTPBTCYZRI_OCESU
```

Within this excerpt, there are 3 unique lines in which the pattern `ZRSNWCFSTPBTCYZRI_OCESU` appears:

* `_JZRSNWCFSTPBTCYZRI_OCESU`  
* `_JOCSI_SSYZRSNWCFSTPBTCYZRI_OCESU`  
* `ZJZRSNWCFSTPBTCYZRI_OCESU_IBTATFK`

Looking at the two characters after each instance of this string, we have the following pairs of two-character sequences:

* `_JZRSNWCFSTPBTCYZRI_OCESU`: `_J`  
* `_JOCSI_SSYZRSNWCFSTPBTCYZRI_OCESU`: `_J`  
* `ZJZRSNWCFSTPBTCYZRI_OCESU_IBTATFK`: `_I`

Now, to apply some rules from the key itself. We know that the Z-coordinate is cycled twice to the right during encryption, and we know that this repeated pattern must always decrypt to the same string. Therefore, when the Z-coordinate is cycled, the Z-coordinate of the last letter in the repeated pattern (`U`) becomes the new Z-coordinate of the character two spaces later; in this case, `J` and `I`.

This implies that `J` and `I` share a Z-coordinate, i.e. `z: JI`. The same principle applies to Y bindings, by taking the character just after the `U` rather than the character after that, however this character is always `_`, and so no Y bindings can be found.

In hindsight, this repeated fragment could actually be extended to include the `_` due to Sebucsert’s wraparound behaviour. However we did not realise this at the time of solving the key, and the supposedly incorrectly discerned bindings from this mistake (`z: JI`) did end up being consistently correct, so it’s being included in our solution anyway.

The same strategy can be applied to the lines that contain the pattern `IZTPNDTPVGNZEECY`:

* `GIZTPNDTPVGNZEECYODC_NXAMUKVAMUSYZRSNXDAWZABTATFKZI_FOWCRQ_YZRKOW_ZSIBTCYZRI_OXPT`: `YO`  
* `GIZTPNDTPVGNZEECYZHS_NXAMUSYZRSTZWKCABTATFKZKPCHCGS_YZRK_RWIQCIBTCYZRI_OXPT`: `YZ`

From this, we can discern `y: OZ, z: DH OZ`.

Similar text patterns exist at the beginning of many lines while the end of that line may vary. This affects only the first two characters at the beginning of the line to vary from the pattern given by the same text at the beginning of a line. By marking down and taking these as bindings we were able to discern all of the relative x bindings and some more of the y bindings than we started with from the PECAN \- ISZID known plaintext. 

```
key_bindings = [
    ["axvoimlpw", "sq_zrcuet", "hjdkbynfg"],
    ["ps", "ezr", "adnkci", "fjy"],
    ["ei", "npz", "acd"]
]
```

## Various plaintext assumptions

- Loki

Using our original bindings, we discovered that the very first character of message.txt consistently decrypts to P. Using a [dictionary](https://www.regexdictionary.com/) and a bit of guesswork, we narrowed this word down to either ‘preface’ or ‘purfler’, with the most likely guess being *preface* given that it was the first word of message.txt

Doing another partial decryption allowed for more visible patterns of words to appear. Near the end of the text we found something that resembles “\_that\_it\_is\_the\_end”. Taking the bindings from this and doing another partial decryption of the text revealed further plaintext.

Bindings after these additions came to:  
```
key_bindings = [
  ['axvoimlpw', 'sq_zrcuet', 'hjdkbynfg'],
  ['ps', 'ezrhw', 'adnkci_g', 'fjytl'],
  ['npzteils', 'acdfw', 'rh', "v_y"]
]
```

## Discovery of Kib ([The Gods of Pegana: The Sayings of Kib | Sacred Texts Archive](https://sacred-texts.com/neu/dun/gope/gope08.htm)) 

- Loki

Stumbling upon *The Sayings of Kib* flipped the challenge on its head. With the bindings inferred from repeated text fragments, we found the plaintext line `kib_is_kib_kib_is_he_and_n!_!ther_!!l!!!` This was more than enough to find *The Gods of Pegana*, a fantasy book by Lord Dunsay from 1905, which we discovered the entirety of message.txt had been copied from. The very first excerpt available on this site provided the bindings for more than enough characters to let us reliably guess the flag.

pecan{clarity\_through\_cubes}

And the final set of bindings:

```
key_bindings = [
  ['q_urtszce', 'ynhkdjgbf', 'wmoavpxil'],`
  ['yqmuvtjlf', 'worhxzgeb', '_nkadspci'],`
  ['y_novtpzb', 'wqmhradcf', 'lukgsxjie']
]
```

## Automated binding extrapolation and combination

- Eclipixie

After the discovery of *The Sayings of Kib*, we quickly developed a couple of functions that could extrapolate and then compress bindings from samples of decrypted and encrypted strings. While this was not immediately useful due to a user error (and loki ended up collapsing enough of the key by hand to discern the flag), these functions were quite useful in assembling the entire key and performing a full decryption on the [entire message file](../resources/sebucsert/message-decrypted.txt).
