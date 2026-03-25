# 3D Point Cloud Rail Alignment Analysis

**Machine learning pipeline for overhead crane rail alignment using laser scanning**

![Python](https://img.shields.io/badge/Python-3.8+-blue.svg)
![scikit-learn](https://img.shields.io/badge/scikit--learn-ML-orange.svg)
![Point Cloud](https://img.shields.io/badge/3D-Point_Cloud-purple.svg)

## Overview

This project develops an automated system for analyzing overhead crane rail alignment using 3D laser scanning data. Traditional rail surveying uses total stations with sparse point measurements - this approach uses dense point clouds (769K+ points) combined with machine learning to:

- **Classify rail components** (head top, head side, web) from raw scan data
- **Extract rail centerlines** with high precision
- **Evaluate CMAA compliance** against industry tolerances (3/16" to 1/4")
- **Identify misalignment locations** requiring adjustment

## Key Results

| Stage | Method | Performance |
|-------|--------|-------------|
| **Classification** | Random Forest | ~98% accuracy on high-density areas |
| **Clustering** | GMM (Gaussian Mixture) | Silhouette score: 0.675 |
| **Alignment** | 20-ft chord analysis | Detected deviations up to 0.311" |

Successfully identified rail sections exceeding CMAA 0.25" tolerance specification.

## Dataset

- **Primary scan**: 425,423 3D points from Leica BLK360 scanner
- **Augmented dataset**: 769,214 points (synthetic fill for low-density gaps)
- **Final analysis**: 265,124 validated points after filtering
- **Coverage**: ~127 feet of overhead bridge crane rails (Rail A & Rail B)

### Point Cloud Features
- X, Y, Z coordinates (converted to feet/inches)
- RGB color values
- Surface normals (Nx, Ny, Nz)
- Illuminance (PCV)

## Technical Approach

### Phase 1: Data Preprocessing
- Coordinate system transformation (meters → feet → inches)
- Rail A/B identification via lateral position thresholding
- Rolling-window quantile analysis for relative coordinates
- Synthetic data augmentation to fill scanner gaps

### Phase 2: Feature Engineering (53 geometric features)
Using PCA and spatial geometry within 1-inch spherical neighborhoods:

| Category | Features |
|----------|----------|
| **Eigenvalue-based** | λ1, λ2, λ3, normalized ratios |
| **Shape descriptors** | Planarity, linearity, sphericity, anisotropy |
| **Surface properties** | Curvature, roughness (mean, RMS, std) |
| **Orientation** | Verticality, dip angle, dip direction |
| **Point cloud metrics** | Neighbor count, illuminance |

### Phase 3: Supervised Classification
- **Training data**: Synthetically-generated ideal rail profile with manual labels
- **Model**: Random Forest classifier
- **Classes**: Rail head top, head side, web, noise
- **Result**: Effective noise filtering and component isolation

### Phase 4: Unsupervised Clustering
Compared 5 clustering methods to isolate rail surfaces:

| Method | Silhouette (Rail A) | Notes |
|--------|---------------------|-------|
| DBSCAN | 0.675 | Best score, eps via k-distance |
| KMeans | 0.652 | Fast, consistent |
| **GMM** | 0.635 | **Selected** - best visual boundaries |
| Spectral | 0.643 | 2-stage approach |
| Ward | 0.639 | Hierarchical with connectivity |

### Phase 5: Alignment Compliance
- Generated averaged polyline at 3-inch stations
- Computed 20-foot chord deviations (CMAA standard)
- Best-fit curve analysis (degree 6 polynomial)
- Identified sections exceeding tolerance

## Project Structure

```
├── rail_1_eda.ipynb                    # Initial data exploration
├── rail_1a_pre_transform_trainer.ipynb # Coordinate transformation
├── rail_2_build_features.ipynb         # 53-feature engineering
├── rail_2a_build_features_trainer.ipynb# Training data features
├── rail_3_components_classification.ipynb    # RF classification
├── rail_3a_components_classification_ideal.ipynb # Ideal profile training
├── KTW C39-42.ipynb                    # Alignment compliance analysis
├── data/
│   ├── imported_data.csv               # Raw point cloud
│   ├── df_processed.csv                # Transformed coordinates
│   ├── df_featured.csv                 # With 53 geometric features
│   └── df_isolated_components.csv      # Classified components
├── rf_classification_ideal_rail.joblib # Trained RF model
└── rf_classification_ideal_rail_metadata.json
```

## Methodology Advantages

Compared to traditional total-station surveying:

| Aspect | Traditional | This Approach |
|--------|-------------|---------------|
| **Point density** | 1 point per 3-5 ft | Thousands per foot |
| **Human error** | Prism leveling variance | Eliminated |
| **Analysis** | Simple linear interpolation | Best-fit curves + ML |
| **Drift** | Accumulated measurement error | Reduced via dense data |

## Technologies Used

- **Python 3.8+**
- **pandas** / **NumPy** - Data manipulation
- **scikit-learn** - Random Forest, clustering algorithms
- **CloudCompare** - Point cloud feature extraction
- **matplotlib** / **seaborn** - Visualization
- **joblib** - Model persistence

## Domain Knowledge

This project applies **CMAA (Crane Manufacturers Association of America)** standards:
- Rail straightness tolerance: 3/16" to 1/4" per 20 feet
- Proper alignment prevents uneven wheel loading, premature wear, and structural failures
- Critical for overhead bridge cranes in industrial facilities

## Future Enhancements

- Automated alignment adjustment recommendations
- Real-time scanning integration
- Additional rail samples for model validation
- Cost-benefit analysis vs. traditional surveying methods