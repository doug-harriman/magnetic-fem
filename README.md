# Finite Element Analysis of Magnetic Systems

This repository is a knowledge base for the finite element analysis of magnetic systems. It is intended to be a collection of resources, tutorials, and examples for the simulation of magnetic fields and forces using finite element analysis (FEA) software.

Personal notes in [Trilium](http://192.168.0.120:32768/#root/P98MayPro1wY/4mDbopUfYitU/m1I8QkAZD0sh?ntxId=SgPeyt).

## 2D Magnetic Analysis

### FEMM

- [FEMM](https://www.femm.info/wiki/HomePage) is a free 2D finite element analysis software for electromagnetic fields. It is a powerful tool for the simulation of magnetic fields and forces in 2D systems.
- FEMM is quite limited in ability to create geometry.  FreeCAD/Elmer is much better for creation of complex geometry.
- [Simplexity internal knowledge base](https://simplexity.atlassian.net/wiki/spaces/SE/pages/2520711717/FEMM+Magnetic+FEA+Analysis).


### FreeCAD/Elmer

* [Excellent how-to video](https://www.youtube.com/watch?v=1ZAmgaqhh_o).
   * [FreeCAD analysis file](./Magnetics-2D.FCStd) generated from the video.
* The default FreeCAD interface to Elmer does not calculate magnetic forces.  See the FreeCAD/Elmer 3D section below for the process to add magnetic force calculation.



## 3D Magnetic Analysis with FreeCAD/Elmer

> [!NOTE]
>
> A full 3D magnetic analysis does not seem to be currently working (as of FreeCAD 1.0, 28-Dec-2025).  See the [issues](#issues) section below.

### Resources

* [Permanent Magnet Simulation with Elmer FEM (Tutorial)](https://www.youtube.com/watch?v=_b0NPP12OCQ)
* [Elmer FEM Webinar - Coil Modeling with ElmerFEM](https://www.youtube.com/watch?v=Z_MBIt1pApU)
    * Covers details about generating a coil & circuit to drive it.
    * [Circuit builder demo code](https://github.com/ElmerCSC/elmer-elmag/tree/main/CircuitBuilder)

A 3D analysis file is [available](./Magnetics-3D.FCStd).  This is the work-in-progress file, and the file used to demonstrate defects submitted to the FreeCAD FEM Workbench team.

### Issues

1. FreeCAD is incorrectly requiring that an Electorstatic Potential constraint be required to run an analysis.
    * [FreeCAD FEM Workbench issue](https://github.com/FreeCAD/FreeCAD/issues/18741)
        * As of 28-Dec-2024, code has been submitted to resolve and release placed in the road map.
    * For workarounds see the issue.

1. FreeCAD is incorrectly failing a check to verify that materials have been applied to all bodies in the analysis.  This as the effect of only allowing one useable body to be analyzed, as one body is typically required for the air surrounding the magnetic system.
    * [FreeCAD FEM Workbench issue](https://github.com/FreeCAD/FreeCAD/issues/18759)
        * As of 28-Dec-2024, the issue has been placed into the road map to be addressed.  No code patches yet.

### Process

1. Create bodies for all magnetic elements (have not tried using parts yet, just bodies)
2. Create body for enclosing volume (air, vacuum, etc.)
3. Select all of the created bodies
4. From the Part Workbench (not Part Design), use the `Boolean Fragments` tool to gather all of the bodies into a Boolean Fragment.
   * This operation cuts the bodies withing the encosing volume out of that volume.  When the Boolean Fragment object is then meshed, this keeps the system from having overlapping meshes.
6. Open the FEB Workbench
7. Create an Analysis Container
8. Delete the default solver in the container.
9. Create an Elmer Solver in the Analysis container.
10. Select the Elmer Solver
11. Add a MagnetoDynamicEquation to the Solver.
12. Select the Analysis Container
13. Add a MaterialFluid (for air)
14. Close out the Task dialog (don't add any Geometry referenes)
15. Select the MaterialFluid again and go to its Data Property View
16. On the references line, click '...'
17. Open the Boolean Fragments container and select the Boolean Fragments object. 
1.  Double click to the right of the Boolean Fragments object.  The field should become editable.
2.  Type in the name of the solid assicated with the enclosing volume.
3.  The macro [mapsolids](https://github.com/doug-harriman/magnetic-fem/blob/main/mapsolids.FCMacro) in this repo attempts to map the Solid names to the associated bodies on the Boolean Fragment.  A propery is added to each Body in the Boolean Fragment with the name of the Solid.  Use this Solid name.
18. Add a MaterialSolid
19. Select Iron-Generic
20. Again in the Data Property View, open the References dialog and manually enter the identifier for the Solids used.  For multiple Solids, separate the values with a comma (`,`).
21. To the Solver, add a Magnetization Boundary Constraint
22. Add a Z magnetization of 5000, click OK.
23. Add the magnet bodies in the same manner as you did for the solid propety.
24. Click the Boolean Fragments container.
25. Click the GMsh icon to create the mesh.
26. Set properties
  * Element Dimension: From Shap
  * Element order: 1st

To-do items:

- [ ] Add in screenshots of each step.
- [ ] Generate a YouTube video demonstrating the process.

### General Learnings

*   Finicky about bodies/solids.  
    *   Got an analysis to run without using a part, just bodies.
*   Mesh → Property View
    *   `Element Order`: 1
        *   At element order 2, got error: “ERROR:: GetEdgeBasis: Can't handle but linear elements, sorry.”
*   Magnetodynamic Solver → Property View
    *   Linear System
        *   `Bi CGstabl Degree` = 4
        *   `Linear Iterative Method` = Bi CGstabl 
        *   `Linear Preconditioning` = None $larr; Unclear if strictly necessary.  Review resources to find source. 
        *   `Linear Solver Type` = Iterative $larr; Unclear if strictly necessary.  Review resources to find source. 
    *   Magnetodynamic
        *   `Angular Frequency` = 0 &larr; Doing magneto static analysis, not high frequency field generation.
    *   Results
        *   `Calculate Magnetic Field Strength` = True
        *   `Calculate Nodal Fields` = True
        *   `Calculate Nodal Forces` = True

### Body Forces

*   Need to add another solver:
    ```
    Solver 4
    	Exec Solver = After All
    	Equation = SaveScalars
    	Procedure = "SaveData" "SaveScalars"
    	Filename = "forces.dat"
    	Variable 1 = nodal force 1
    	Operator 1 = boundary sum
    	Variable 2 = nodal force 2
    	Operator 2 = boundary sum
    	Variable 3 = nodal force 3
    	Operator 3 = boundary sum
    End
    ```

*   Need the "Variable N" and “Operator N” pair for each numbered body in the file.

*   Number of the solver, in this case 4, should be 1 more than the highest number in the `.sif` file before adding this one.

*   Replace any instances of something like:

       `Active Solvers(3) = Integer 1 2 3`

    with:

    `Active Solvers(4) = Integer 1 2 3 4`

    where the ‘4’ is the number of the solver we just added.

> [!NOTE]
>
>  Due to the way FreeCAD runs Elmer, it will overwrite the input `.sif` file if edited externally.  

As of FreeCAD 1.0, you must do the following:

1. Write the file per the Elmer Solver Task Pane
1. Click the Edit button in that Pane.
1. Edit the file.
1. Run the analysis.
