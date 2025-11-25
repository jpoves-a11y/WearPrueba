# Acetabular Wear Analysis System

## Overview
This project is a professional standalone HTML web application designed for the precise analysis of volumetric and linear wear in acetabular hip prosthesis components. It processes STL files to provide 3D visualization, automated wear detection, and comprehensive measurement capabilities. The application's core purpose is to calculate volumetric wear, measure linear penetration depth, visualize wear patterns with color-coded 3D rendering, and export analysis results for medical documentation, contributing to improved medical diagnostics and prosthesis design.

## Recent Changes
- **2025-11-25 (v3.2) - Hemispheric Asymmetry Analysis**: Completely redesigned wear detection using simple, robust hemispheric comparison. PREVIOUS PROBLEM: v3.0-3.1 algorithms were too conservative and still detected center as worn. NEW ALGORITHM (v3.2 - SIMPLE & DIRECT): (1) **Geometric Centroid**: Calculates centroid of inner surface (not sphere center). (2) **Multi-Axis Testing**: Tests 8 different division axes (X, Y, Z, and diagonals) to find maximum asymmetry. (3) **Hemispheric Distance Comparison**: For each axis, divides surface into two hemispheres and calculates average distance to centroid for each hemisphere. (4) **Asymmetry Detection**: Selects the axis with maximum distance difference between hemispheres. (5) **Simple Classification Rule**: Hemisphere CLOSER to centroid = WORN (laterally displaced inward). Hemisphere FARTHER from centroid = UNWORN (maintains original position). (6) **Inflection Points**: All vertices at the hemispheric boundary (worn vertices with unworn neighbors and vice versa). (7) **Key Insight**: Lateral displacement creates clear hemispheric asymmetry - one side closer, one side farther. NO scoring functions, NO thresholds, just pure geometric comparison. Typical distribution: ~50/50 hemispheric split. MUCH SIMPLER than v3.1.
- **2025-11-25 (v3.0.1) - Stack Overflow Fix**: Fixed "Maximum call stack size exceeded" error in wear detection algorithm. Replaced spread operator (`Math.max(...array)`) with iterative loops for calculating min/max values of large vertex arrays (205k+ vertices). This error was reintroduced in v3.0 algorithm and has now been corrected. The rim vertex detection step (lines 1882-1898) now uses O(n) iterative loops instead of spread operators to calculate the principal axis spread, preventing stack overflow with large datasets.
- **2025-11-25 (v3.0) - Major Algorithm Upgrade**: Completely redesigned wear detection algorithm to recognize "spherical displacement" patterns characteristic of femoral head wear. NEW ALGORITHM: (1) **Spherical Deviation Analysis**: Fits preliminary reference sphere to entire inner surface, then calculates local deviations (magnitude and direction vectors) for each vertex. (2) **Local Consistency Scoring**: Unworn zones maintain near-zero deviation (spherical), worn zones show consistent positive deviation in aligned directions (displacement pattern). Score combines deviation magnitude, neighbor consistency, and vector alignment. (3) **Automatic Inflection Point Detection**: Identifies transition boundary vertices (worn vertices with unworn neighbors + unworn vertices with worn neighbors) automatically without manual thresholds. (4) **Intelligent Classification**: Uses percentile-based thresholding (Q75 + 0.5*IQR) to classify 15-20% as worn (minority), 80-85% as unworn (majority). (5) **Benefits**: More accurate recognition of displacement patterns, robust to wear orientation, automatic boundary detection, computationally efficient for ~200k vertices. Algorithm specifically designed per user requirements to detect the characteristic "one-sided spherical displacement" caused by femoral head contact.
- **2025-11-25**: Replit environment import completed. (1) Installed Python 3.11 module for server runtime. (2) Enhanced server.py with SO_REUSEADDR socket option via custom ReuseAddressTCPServer class to prevent "Address already in use" errors on workflow restarts. (3) Created attached_assets/ directory and properly organized favicon file. (4) Configured "Start application" workflow to serve on 0.0.0.0:5000 with webview output type for frontend preview. (5) Configured static site deployment for production publishing. (6) Created .gitignore file with Python and IDE exclusions. (7) Application is fully functional and verified with screenshot testing.
- **2025-11-24 (v2.5)**: Enhanced export capabilities and UI improvements. (1) Updated Legend with all visual elements: added Inflection Markers (yellow spheres), Sphere Surface Point (cyan marker), and Deepest Worn Point (magenta marker) for complete visualization reference. (2) Implemented full PDF export using jsPDF library - generates professional reports with all wear metrics, sphere parameters, fitting quality diagnostics, zone areas, and transition plane data. (3) Added Interactive HTML export feature that creates standalone 3D visualization files with embedded Three.js - includes complete geometry, fitted sphere, transition plane, and wear data with orbit controls for sharing and offline viewing. (4) All export buttons now fully functional with proper error handling and user feedback.
- **2025-11-24 (v2.4)**: CRITICAL FIX for zero-results issue. Corrected transition plane normal orientation - now points INWARD (from boundary toward sphere center) instead of outward. This ensures worn vertices (closer to center) correctly pass the distance <= 0 filter. Added comprehensive Spanish diagnostic alerts that automatically identify the root cause when results are zero (no worn zones detected, plane exclusion, vertex filtering, or negative penetration). Updated all documentation and code comments to reflect correct plane orientation.
- **2025-11-24 (v2.2)**: Critical fix for transition plane calculation. Corrected boundary vertex detection to find inflection points on BOTH sides of the worn/unworn interface (unworn vertices with worn neighbors AND worn vertices with unworn neighbors). Replaced PCA method with more robust RADIAL method: plane normal is the radial direction from sphere center to boundary centroid, and plane is positioned at the centroid of all inflection points. This ensures the transition plane passes directly through the true inflection zone.
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
-   **Worn/Unworn Zone Detection (v3.0 - NEW)**: Uses **Spherical Displacement Recognition Algorithm** specifically designed to detect the characteristic "one-sided displacement" pattern caused by femoral head wear. Steps: (1) Fits preliminary reference sphere to entire inner surface (2) Calculates spherical deviation (magnitude + direction vector) for each vertex (3) Computes local consistency scores combining own deviation, neighbor deviation ratio, and vector alignment (4) Classifies using percentile thresholds: unworn = low deviation + low consistency (majority, 80-85%), worn = high deviation + high consistency (minority, 15-20%) (5) Automatically detects inflection points at worn/unworn boundary for transition plane. Automatically excludes rim/edge vertices (top 12% radial distance + boundary vertices with <4 neighbors).
-   **Unworn Sphere Fitting**: Two robust methods available:
    - **Gauss-Newton + LM**: Fast convergence with Levenberg-Marquardt damping for stable optimization
    - **RANSAC + LM**: Robust consensus-based fitting that handles outliers, followed by LM refinement on inliers
-   **Transition Plane Detection**: Uses RADIAL method to compute the transition plane at the worn/unworn interface. Boundary detection algorithm identifies inflection points on BOTH sides of the interface (unworn→worn and worn→unworn transitions) to ensure complete boundary representation. The plane is positioned at the centroid of all boundary points, with normal vector pointing INWARD (from boundary centroid toward sphere center) to correctly delimit the worn zone. This ensures worn vertices (closer to center) have negative/zero signed distance and pass the filter correctly. Includes quality metric (average distance from boundary to fitted plane).
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
-   **Server Configuration**: Custom Python HTTP server (server.py) with:
    - SO_REUSEADDR socket option for graceful restarts
    - Cache-Control headers (no-cache, no-store, must-revalidate) to prevent browser caching
    - Custom logging for cleaner output
-   **Deployment**: Configured as static site deployment (serves index.html and all assets from root directory)
-   **Asset Management**: Favicon and generated files stored in attached_assets/ directory
-   **No Build Required**: Standalone HTML application with all dependencies loaded via CDN