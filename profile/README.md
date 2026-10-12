# .github

# Transcriptomic Pipeline for ESCC

Welcome to the **Transcriptomic Pipeline** organization. This workspace hosts a end-to-end bioinformatics pipeline developed specifically for the analysis of transcriptome sequencing (RNA-seq) data from patients with cancer.

**Project Focus:** *To develop an automated pipeline for the analysis of transcriptome sequencing data of patients with ESCC.*

**Author:** Babul Pradhan, Ph.D. Student

**Supervisor:** Dr. Jyoti Sharma

**Affiliations:** Manipal Academy of Higher Education (MAHE), Manipal, India & Institute of Bioinformatics (IOB), Bangalore, India

---

## 🧬 Architectural Overview

The pipeline is engineered for unattended, high-throughput execution. To ensure strict environmental reproducibility and regulatory compliance, the workflow is distributed across six specialized analytical modules and one master module to call different molecular alterations among gene fusion, differential gene expressions, alternate splicing events, and transcript variants.

Each module operates independently and resolves its own dependencies (via Conda, Singularity/Apptainer, or local compilation), preventing version conflicts across the analytical graph.

Any specific module(s) can run independently using the respective modules or as end to end module.

### The Master Orchestrator

* 🚀 **[e2e-orchestrator](https://github.com/transcriptomic-pipeline/e2e)**: The master execution wrapper. It utilizes a single `pipeline_master.conf` file to enforce global parameters and reference paths. It dynamically parses a base clinical manifest and automatically translates it into the highly specific sub-maps required by downstream tools, bridging raw inputs to final outputs without manual intervention.

### The Analytical Modules

1. **[qc](https://github.com/transcriptomic-pipeline/qc)**: Raw sequencing data evaluation and read truncation using **FastQC** and **Trimmomatic**. Handles adapter clipping and low-quality base removal to prepare reads for splice-aware alignment.
2. **[alignment](https://github.com/transcriptomic-pipeline/alignment)**: Splice-aware mapping to the GRCh38 human reference genome using **STAR** (for maximum splice detection sensitivity) or **HISAT2** (for memory-constrained environments), outputting coordinate-sorted BAM files.
3. **[fusion](https://github.com/transcriptomic-pipeline/fusion)**: Dual-caller oncogenic fusion gene detection utilizing **STAR-Fusion** and **Arriba**. Includes a rigorous post-processing intersection script that applies DepMap filters to eliminate known artifactual and read-through transcripts.
4. **[dge](https://github.com/transcriptomic-pipeline/dge)**: Differential Gene Expression quantification using **featureCounts**, followed by statistical modeling, size factor normalization, and dispersion estimation using **DESeq2** in R.
5. **[alt-splicing](https://github.com/transcriptomic-pipeline/alt-splicing)**: Identification of tumor-specific alternative splicing events (skipped exons, retained introns, mutually exclusive exons) using **rMATS**, enhanced with Python-based recurrence and $\Delta\Psi$ filtering.
6. **[transcript-variants](https://github.com/transcriptomic-pipeline/transcript-variants)**: Somatic SNV and INDEL calling via the **GATK Best Practices** framework. Automates BAM preprocessing (Picard MarkDuplicates, SplitNCigarReads, BQSR) and executes **Mutect2** utilizing a Panel of Normals (PoN) and the gnomAD population database to isolate true somatic mutations.

---

## 📊 Pipeline Data Flow

The orchestrator dynamically routes data through the directed acyclic graph (DAG) based on user configuration.text
┌───────────────────────────┐
│    Raw FASTQ Directory    │
└─────────────┬─────────────┘
│
[ Module: qc ]
│
┌─────────────┴─────────────┐
│   Trimmed FASTQ Output    │
└──────┬──────────────┬─────┘
│              │
[ Module: alignment ]   [ Module: fusion ]  [ Module: dge ]
│              │
┌──────────────┴───────────┐  │
│  Coordinate-Sorted BAMs  │  │
└──────┬────────────┬──────┘  │
│            │         │
┌──────────────┼────────────┴─────────┼──────────────┐
│              │                      │              │
[ Module: dge ] [ Module: alt-splicing ] [ Module: variants ]
│              │                      │              │
┌─────┴─────┐  ┌─────┴─────┐          ┌─────┴─────┐  ┌─────┴─────┐
│ DEGs List │  │ Splicing  │          │ Somatic   │  │ Fusions   │
│ & Plots   │  │ Events    │          │ VCFs      │  │ Summary   │
└───────────┘  └───────────┘          └───────────┘  └───────────┘

```

---

## ⚙️ System Requirements & Deployment

Due to the computational demands of splice-aware human genome alignment and fusion calling, the following environment is recommended:

*   **Operating System**: Linux (Ubuntu 20.04+ or CentOS/RHEL 8+)
*   **Memory**: $\ge$ 32 GB RAM (64 GB strictly recommended for concurrent STAR operations)
*   **Storage**: $\ge$ 1 TB NVMe or SSD storage for intermediate BAM files and reference indexes
*   **Dependencies**: Git, Bash (POSIX compliant), Wget. The `install_e2e.sh` script handles the downstream resolution of Conda, Python 3, Java, Perl, and Apptainer/Singularity.

### Quick Start
To initialize the workspace and run the full pipeline:

1. Clone the master end to end module into your `opt` or workspace directory:
   ```bash
   git clone [https://github.com/transcriptomic-pipeline/e2e.git](https://github.com/transcriptomic-pipeline/e2e.git)
   cd e2e

```

2. Run the deployment script to pull all 6 modules and their respective tools:
```bash
bash install_e2e.sh

```


3. Edit the `pipeline_master.conf` to point to your raw FASTQ directory and reference genomes.
4. Execute the unattended run:
```bash
bash run_e2e.sh pipeline_master.conf

```



---

## 📄 License & Academic Citation

The codebase across all repositories in this organization is released under open-source licenses (MIT/CC0).

If you utilize this architectural framework or any of its constituent modules in your research, please provide attribution:

> **Pradhan, B., & Sharma, J. (2025).** *Automated Transcriptomic Analysis Pipeline for Esophageal Squamous Cell Carcinoma*. Institute of Bioinformatics (IOB), Bangalore. GitHub: [transcriptomic-pipeline](https://github.com/transcriptomic-pipeline).

```

```
