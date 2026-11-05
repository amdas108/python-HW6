# HW6 — Virtual Environments

## Environment

The script was tested on the following system:

- **OS:** macOS 26.0 (Apple Silicon)
- **Python:** 3.14.10

## 1. Create and activate a virtual environment

Clone the repository and navigate to the project directory:

```bash
git clone https://github.com/amdas108/python-HW6.git
cd python-HW6
```

Create and activate a virtual environment:

```bash
python -m venv venv
source venv/bin/activate
```

Upgrade pip:

```bash
python -m pip install --upgrade pip
```

## 2. Install dependencies

Install all required packages:

```bash
python -m pip install -r requirements.txt
```

## 3. Restore the legacy Biopython API

The script imports `Bio.SubsMat.MatrixInfo`, which is no longer available in Biopython 1.88.

In order to run the `pain.py`, recreate the legacy module using the substitution matrices provided by the installed Biopython version (just copy-paste and run in your terminal):

```bash
BIO_DIR=$(python -c "import Bio, pathlib; print(pathlib.Path(Bio.__file__).parent)")

mkdir -p "$BIO_DIR/SubsMat"

cat > "$BIO_DIR/SubsMat/__init__.py" <<'PY'
from . import MatrixInfo
PY

cat > "$BIO_DIR/SubsMat/MatrixInfo.py" <<'PY'
from Bio.Align import substitution_matrices

_matrix = substitution_matrices.load("BLOSUM62")

blosum62 = {
    (a, b): float(_matrix[a, b])
    for a in _matrix.alphabet
    for b in _matrix.alphabet
}
PY
```

This compatibility module restores the old `MatrixInfo.blosum62` interface using the actual BLOSUM62 substitution matrix.

## 4. Patch pandas compatibility

The script passes a `set` as the DataFrame index. Pandas 2.2.3 explicitly rejects sets as index values.

To preserve the original script and make the index deterministic, replace the exception with sorting of the set in `pandas/core/frame.py` (just copy-paste and run in your terminal):

```bash
PANDAS_FRAME=$(python -c "import pandas.core.frame as f; print(f.__file__)")

python - "$PANDAS_FRAME" <<'PY'
import sys
from pathlib import Path

file_path = Path(sys.argv[1])
content = file_path.read_text()

old_code = '            raise ValueError("index cannot be a set")'
new_code = '            index = sorted(index)'

if new_code in content:
    print("Pandas compatibility patch is already applied.")
elif old_code in content:
    content = content.replace(old_code, new_code, 1)
    file_path.write_text(content)
    print("Pandas compatibility patch applied.")
else:
    raise SystemExit("Could not find the expected pandas source code.")
PY
```

The patch converts the set into a sorted list before constructing the DataFrame.

## 5. Run the script

After completing the steps above, run:

```bash
python pain.py
```

If everything works correctly, you should see the following message:

```bash
✨❤ Всё работает, ты больш(ой/ая) молодец!!! ✨❤
```