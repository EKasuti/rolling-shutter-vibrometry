# Final project

**Course:** COSC 073 - Computational Photography  
**Term:** Spring 2026  
**Author:** Kasuti Makau

---

# Data

Full dataset on Google Drive: **[[Link](https://drive.google.com/drive/folders/1QFCNzPZNQcSuhTbEX7EZb-97NZC-FhF_?usp=sharing)]**


---

## Getting Started

1. Create a virtual environment:
   ```bash
   python3 -m venv venv
   ```

2. Activate the virtual environment:
   ```bash
   source venv/bin/activate
   ```

3. Install requirements:
   ```bash
   pip install -r requirements.txt
   ```

4. Register the venv as a Jupyter kernel:
   ```bash
   pip install ipykernel
   python -m ipykernel install --user --name=final-project --display-name "Python (final-project)"
   ```

### VS Code kernel setup

1. Open `final.ipynb` in VS Code.
2. Click **Select Kernel** (top right of the notebook).
3. Choose **Python Environments** -> select **Python (final-project)** (the kernel registered above).
4. If it doesn't appear, press `Cmd+Shift+P` -> **Python: Select Interpreter** -> point it to `./venv/bin/python`, then reopen the kernel picker.

---