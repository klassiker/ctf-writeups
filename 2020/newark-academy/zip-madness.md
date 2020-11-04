# Zip Madness

## Task

Evan is playing Among Us and just saw an imposter vent in front of him! Help him get to the emergency button by following the directions at each level.

File: flag.zip

## Solution

This sounds like a recursive zip and yes, it is.

```bash
mkdir -p zips
unzip flag.zip -o -d zips

for i in $(seq 1000 -1 1);do
  d=$(cat zips/direction.txt)
  if [[ "$d" == "left" ]];then
    unzip -o "zips/${i}left.zip" -d zips
  elif [[ "$d" == "right" ]];then
    unzip -o "zips/${i}right.zip" -d zips
  else
    echo "Unknown direction encountered at ${i}"
    exit
  fi
done

cat zips/flag.txt
```

And we get: `nactf{1_h0pe_y0u_d1dnt_d0_th4t_by_h4nd_87ce45b0}`
