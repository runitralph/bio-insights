# Bio Insights: Analysis of proteins with known secondary structures and transmembrane regions

This project demonstrates the ability to handle biological data, use SQL for data storage, and perform basic bioinformatics analysis using Python. It showcases skills in database management, data retrieval, and data analysis showing example code snippets.

## Introduction

The proteins analysed were selected from a compilation created during my BSc and MSc projects, in which I evaluated the accuracy of bioinformatics tools for predicting the secondary structure and transmembrane regions of proteins. Protein structure determination often requires time-intensive techniques like X-ray crystallography and NMR spectroscopy. This led to the emergence of the structure-sequence gap, underscoring the significance of predictive tools like Google DeepMind's innovative [AlphaFold](https://alphafold.ebi.ac.uk/) and the [AlphaFold Server](https://alphafoldserver.com/). In my projects, proteins with known structures were identified using their PDB IDs, and their amino acid sequences were obtained in FASTA format from a protein sequence database. These FASTA sequences were then processed through various online predictive tools, and statistical analyses were conducted to assess the performance of these tools.

This project repo aims to demonstrate the application of modern tools such as MySQL and Python in the analysis of established protein sequence data.

## Code Snippets

### SQL: Building a protein database - [protein_db.sql](https://github.com/runitralph/script-sandbox/blob/main/protein_db.sql)

The following MySQL code was used to create a `proteins` table to store the protein data:

```sql

CREATE TABLE proteins (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    sequence TEXT NOT NULL,
    description TEXT
);
```
### Python: Updating the database and analysing proteins - [protein_db_analysis.py](https://github.com/runitralph/script-sandbox/blob/main/protein_db_analysis.py)

Various python functions were called to read csv files containing all PDB IDs, fetch FASTA sequences from the RCSB, parse the FASTA sequences, and insert the sequences alongside PDB IDs into the MySQL table created above.

Dependencies:
```python
import os
import csv
import requests
import mysql.connector
```
Using PDB IDs to fetch FASTA sequences:
```python
def fetch_fasta(pdb_id):
    url = f'https://www.rcsb.org/fasta/entry/{pdb_id}'
    response = requests.get(url)
    if response.status_code == 200:
        return response.text
    else:
        print(f"Failed to fetch data for PDB ID {pdb_id}")
        return None
```
Parsing FASTA sequences:
```python
def parse_fasta(fasta_text):
    sequences = []
    sequence = ''
    name = ''
    description = ''
    for line in fasta_text.split('\n'):
        if line.startswith('>'):
            if sequence:
                sequences.append((name, sequence, description))
            header = line[1:].strip().split(None, 1)
            name = header[0]
            description = header[1] if len(header) > 1 else ''
            sequence = ''
        else:
            sequence += line.strip()
    if sequence:
        sequences.append((name, sequence, description))
    return sequences
```
Insert protein sequences into MySQL database:
```python
def insert_proteins(sequences):
    conn = mysql.connector.connect(
        host="host_name",
        user="uname_here",
        password="pass_here",
        database="protein_db"
    )
    cur = conn.cursor()
    for name, sequence, description in sequences:
        cur.execute(
            "INSERT INTO proteins (name, sequence, description) VALUES (%s, %s, %s)",
            (name, sequence, description)
        )
    conn.commit()
    cur.close()
    conn.close()
```
Retrieving and analysing protein sequences from MySQL database, and writing output to `protein_analysis.csv`:
```python
def get_proteins():
    conn = mysql.connector.connect(
        host="host_name",
        user="uname_here",
        password="pass_here",
        database="protein_db"
    )
    cur = conn.cursor()
    cur.execute("SELECT id, name, sequence FROM proteins")
    proteins = cur.fetchall()
    cur.close()
    conn.close()
    return proteins

def calculate_molecular_weight(sequence):
    weights = {'A': 89.1, 'C': 121.2, 'D': 133.1, 'E': 147.1, 'F': 165.2,
               'G': 75.1, 'H': 155.2, 'I': 131.2, 'K': 146.2, 'L': 131.2,
               'M': 149.2, 'N': 132.1, 'P': 115.1, 'Q': 146.2, 'R': 174.2,
               'S': 105.1, 'T': 119.1, 'V': 117.1, 'W': 204.2, 'Y': 181.2}
    molecular_weight = 0
    for aa in sequence:
        if aa in weights:
            molecular_weight += weights[aa]
        else:
            print(f"Ignoring unknown amino acid: {aa}")
    return molecular_weight

def analyse_proteins():
    proteins = get_proteins()
    with open('protein_analysis.csv', 'w', newline='') as csvfile:
        fieldnames = ['Protein ID', 'Name', 'Molecular Weight']
        writer = csv.DictWriter(csvfile, fieldnames=fieldnames)
        writer.writeheader()
        for prot_id, name, sequence in proteins:
            weight = calculate_molecular_weight(sequence)
            writer.writerow({'Protein ID': prot_id, 'Name': name, 'Molecular Weight': weight})
```

## Resources

### Protein structure prediction
* [AlphaFold Repository](https://github.com/google-deepmind/alphafold)
* [RoseTTAFold Repository](https://github.com/RosettaCommons/RoseTTAFold)
