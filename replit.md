# Acetabular Wear Analysis System

## Overview
This project is a professional standalone HTML web application designed for the precise analysis of volumetric and linear wear in acetabular hip prosthesis components. It processes STL files to provide 3D visualization, automated wear detection, and comprehensive measurement capabilities. The application's core purpose is to calculate volumetric wear, measure linear penetration depth, visualize wear patterns with color-coded 3D rendering, and export analysis results for medical documentation, contributing to improved medical diagnostics and prosthesis design.

## Recent Changes
- **2025-11-24**: Replit environment setup. Configured Python 3.11 HTTP server (server.py) to serve the static HTML application on 0.0.0.0:5000 with proper cache control headers (Cache-Control: no-cache, no-store, must-revalidate). Set up workflow for automatic server startup. Configured deployment as static site for production publishing.
- **2025-11-23 (v2.1)**: Critical bug fixes and new features. Fixed transition plane orientation bug in PCA calculation by replacing incorrect power iteration (which found largest eigenvalue/in-plane direction) with cross-product method that correctly computes the plane normal from boundary points. Added "Unworn Area Match" metric to Quality Diagnostics panel showing percentage of unworn vertices matching the fitted sphere within 2×RMS tolerance. Implemented comprehensive download functionality for Reference Sphere Visualization elements, exporting 5 files (inner_surface.stl, fitted_sphere.stl, transition_plane.stl, inflection_points.csv, sphere_view_metadata.json) for external CAD/analysis tools. Fixed all "Maximum call stack size exceeded" errors by replacing spread operators and .map() calls with iterative loops for large vertex arrays (313k+ unworn, 41k+ worn vertices).
- **2025-11-23 (v2.0)**: Major enhancement release. Implemented modular ES6 architecture with specialized services. Added RANSAC + LM sphere fitting for robustness against outliers. Replaced transition plane calculation with PCA (Principal Component Analysis) for better accuracy. Implemented comprehensive quality diagnostics (RMS error, residual distribution, inlier count). Added interactive visualization of inflection points with editable markers. Enhanced JSON/CSV export with complete metadata including fitting parameters, error statistics, and plane coordinates. Added quality validation UI panel showing all diagnostic metrics.
- **2025-11-21**: Corrected linear wear calculation to properly frame measurements within the worn zone defined by the transition plane. The algorithm now filters worn vertices to include only those on the worn side of the transition plane (distance ≤ 0), ensuring measurements accurately represent the space between the transition plane and the real inner surface of the prosthesis.

## User Preferences
- Language: English (all UI and outputs)
- Professional medical-grade interface required
- Standalone HTML preferred (no build step)
- Clear visual distinction between worn/unworn zones to prevent confusion

## System Architecture

### UI/UX Decisions
The application features a professional medical-grade user interface built with Tailwind CSS, utilizing a clean white background and a modern color scheme. A dual-viewer system provides simultaneous visualization of the main analysis and the fitted reference sphere. Color-coding (Blue for original, Green for unworn, Red for worn, Gold wireframe for reference sphere) ensures clear visual distinction and intuitive interpretation of results. The interface is designed around a sequential 4-step analysis pipeline, guiding the user through the process.

### Technical Implementations
The system is implemented as a standalone HTML5 application with modular ES6 architecture, leveraging Three.js for 3D rendering and scientific visualization, with all dependencies loaded via CDN for a zero-build-step deployment. It supports both binary and ASCII STL file formats. The codebase is organized into specialized service modules:

**Modular Architecture (v2.0):**
-   **GeometryService**: Core geometric operations (adjacency building, triangle area calculation, angle computations)
-   **CurvatureAnalyzer**: Discrete differential geometry using Meyer et al. 2003 method for Gaussian curvature and mixed Voronoi areas
-   **PCAService**: Principal Component Analysis for optimal plane fitting from point clouds with weighted centroids
-   **FittingService**: Multiple sphere fitting strategies with comprehensive diagnostics

**Key Algorithms:**
-   **Inner Surface Isolation**: Uses normal vector analysis combined with robust connected component filtering to accurately isolate the concave inner surface of the acetabular cup, eliminating spurious regions.
-   **Worn/Unworn Zone Detection**: Employs Gaussian curvature analysis (Meyer et al. 2003) for wear zone identification, detecting saddle points characteristic of femoral head wear. This is combined with radial deviation metrics and an automatic rim/edge exclusion algorithm to precisely delineate worn (red) and unworn (green) zones.
-   **Unworn Sphere Fitting**: Two robust methods available:
    - **Gauss-Newton + LM**: Fast convergence with Levenberg-Marquardt damping for stable optimization
    - **RANSAC + LM**: Robust consensus-based fitting that handles outliers, followed by LM refinement on inliers
-   **Transition Plane Detection**: Uses PCA (Principal Component Analysis) to compute the optimal plane fitting all boundary points between worn and unworn regions. PCA minimizes variance perpendicular to the plane, providing a statistically robust boundary definition. Includes quality metric (average distance from boundary to fitted plane).
-   **Volumetric Wear Calculation**: Calculates volumetric wear ONLY in the worn zone as the integral volume between the fitted sphere (ideal surface) and the real worn surface. For each worn triangle, vertices are projected onto the fitted sphere and the wedge volume between real and ideal surfaces is computed using tetrahedral decomposition from the sphere center.
-   **Linear Wear Metrics**: Calculates mean, max, and min perpendicular penetration depths from the fitted unworn sphere to the real inner surface. Measurements are framed within the worn zone defined by the transition plane, only including vertices on the worn side (between the transition plane and the inner surface).
-   **Quality Diagnostics**: Comprehensive fitting diagnostics including RMS error, iteration count, inlier/outlier ratio, residual distribution (min/max/mean), and convergence status.

### Feature Specifications
The application pipeline consists of four main steps:
1.  Isolate Inner Surface
2.  Detect Wear Zones
3.  Fit Reference Sphere (with method selection: Gauss-Newton or RANSAC)
4.  Calculate Wear

**Interactive Features:**
-   Dual 3D viewers with orbit controls
-   Sphere fitting method selector (Gauss-Newton vs RANSAC)
-   Interactive inflection point markers (toggleable visualization)
-   Real-time quality diagnostics panel
-   Color-coded wear zones with transparency

**Export Capabilities:**
-   **CSV Export**: Tabular data with all key metrics
-   **JSON Export (Enhanced)**: Complete metadata including:
    - Wear metrics (volumetric and linear)
    - Sphere parameters and fitting diagnostics
    - Transition plane equation and PCA method
    - Quality metrics (RMS error, residuals, inlier ratio)
    - Curvature analysis parameters
    - Inflection point statistics
    - Analysis timestamp and system version

### System Design Choices
The application is designed for performance, utilizing direct manipulation of BufferGeometry attributes for efficient rendering and O(n) algorithms where possible. The architecture prioritizes a standalone, client-side approach for ease of deployment and accessibility.

## External Dependencies
-   **3D Rendering Library**: Three.js (v0.158.0) via CDN
-   **UI Framework**: Tailwind CSS via CDN
-   **Development Server**: Python 3.11 HTTP server with cache control headers (server.py)

## Replit Environment Setup
-   **Development**: Run the "Start application" workflow to serve on http://0.0.0.0:5000/
-   **Deployment**: Configured as static site deployment (serves index.html and all assets from root directory)
-   **Cache Control**: Custom HTTP server prevents browser caching issues during development
-   **No Build Required**: Standalone HTML application with all dependencies loaded via CDN