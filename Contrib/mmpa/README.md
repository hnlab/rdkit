# MMPA

## fragmentation
```bash
python rfrag.py < data/sample.smi > data/sample_fragmented.txt

head -n 3 data/sample_fragmented.txt
#
# Cc1cc(C)n(Cc2ccc(C(=O)O)o2)n1,658387,Cc1cc([*:1])nn1[*:2],C[*:1].O=C(O)c1ccc(C[*:2])o1
# Cc1cc(C)n(Cc2ccc(C(=O)O)o2)n1,658387,,C[*:1].Cc1cc([*:1])n(Cc2ccc(C(=O)O)o2)n1
# format: WHOLE_MOL_SMILES,ID,SMILES_OF_CORE,SMILES_OF_CONTEXT
```

## indexing
```bash
python indexing.py < data/sample_fragmented.txt > data/sample_mmps_default.csv

head -n 3 data/sample_mmps_default.csv
# Cc1cc(C)n(Cc2ccc(C(=O)O)o2)n1,Cc1cc(C)n(Cc2ccc(C(=O)O)cc2)n1,658387,615212,c1cc([*:1])oc1C[*:2]>>c1cc([*:1])ccc1C[*:2],Cc1cc(C)n([*:2])n1.O=C(O)[*:1]
# Cc1cc(C)n(Cc2ccc(C(=O)O)o2)n1,Cc1cc(C)n(Cc2ccc(C(=O)O)cc2)n1,658387,615212,c1cc([*:1])oc1[*:2]>>c1cc([*:1])ccc1[*:2],Cc1cc(C)n(C[*:1])n1.O=C(O)[*:2]
# Cc1cc(C)n(Cc2ccc(C(=O)O)o2)n1,Cc1cc(C)n(Cc2ccc(C(=O)O)cc2)n1,658387,615212,O=C(O)c1ccc(C[*:1])o1>>O=C(O)c1ccc(C[*:1])cc1,Cc1cc(C)n([*:1])n1
# format: SMILES_OF_LEFT_MMP,SMILES_OF_RIGHT_MMP,ID_OF_LEFT_MMP,ID_OF_RIGHT_MMP,SMIRKS_OF_TRANSFORMATION,SMILES_OF_CONTEXT
```

## create_db
```bash
python create_mmp_db.py -s < data/sample_fragmented.txt

du -sh mmp.db
# 56K     mmp.db
```

## search_db

**Only support string match. it can't support smiles or smarts match.**

### 1. mmp
Find all MMPs of a input/query compound to the compounds in the db
```bash
cat data/sample_db_input_smi.txt
# c1cc2c(ncnc2NCc2cccnc2)s1 2531831

python search_mmp_db.py -t mmp < data/sample_db_input_smi.txt
# c1cc2c(ncnc2NCc2cccnc2)s1,c1cc2c(ncnc2NCc2ccccc2)s1,2531831,2139597,c1ccc([*:1])cc1,c1nc(NC[*:1])c2ccsc2n1
```

### 2. subs
Find all MMPs in the db where the LHS of the transform matches an input
substructure.
```bash
cat data/sample_db_input_subs.txt
# [*]c1ccccc1

python search_mmp_db.py -t subs < data/sample_db_input_subs.txt
# no result!
```

### 3. trans
Find all MMPs that match the input transform/SMIRKS. Make sure the input SMIRKS has been canonicalised using the cansmirk.py program.

```bash
cat data/sample_db_input_trans.txt
# [*:1]c1ccccc1>>[*:1]c1cccnc1,t1
python search_mmp_db.py -t trans < data/sample_db_input_trans.txt
# no result!

tail -n 1 data/sample_mmps_default.csv
# c1cc2c(ncnc2NCc2cccnc2)s1,c1cc2c(ncnc2NCc2ccccc2)s1,2531831,2139597,c1cncc([*:1])c1>>c1ccc([*:1])cc1,c1nc(NC[*:1])c2ccsc2n1

echo 'c1cncc([*:1])c1>>c1ccc([*:1])cc1' > /tmp/trans.txt
python search_mmp_db.py -t trans < /tmp/trans.txt
# c1cc2c(ncnc2NCc2cccnc2)s1,c1cc2c(ncnc2NCc2ccccc2)s1,2531831,2139597,c1cncc([*:1])c1>>c1ccc([*:1])cc1,c1nc(NC[*:1])c2ccsc2n1

```

