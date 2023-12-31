# SAMLfD
Implementation of Similarity-Aware Multi-representational Learning from Demonstration (SAMLfD) framework using Python.
 
Mantained by Brendan Hertel (brendan_hertel@student.uml.edu).
 
This code follows the implementation proposed by the paper "Similarity-Aware Skill Reproduction based on Multi-Representational Learning from Demonstrations" by Brendan Hertel and Reza Ahmadzadeh, which can be seen here : https://arxiv.org/abs/2110.14817
 
An image of the framework process can be seen below:

![Framework](https://github.com/brenhertel/Meta-LfD/blob/master/.media/mlfd_flowchart_updated_3d_v2.png)

If you use the code present in this repository, please cite the following paper:
```
@inproceedings{hertel2021SAMLfD,
  title={Similarity-Aware Skill Reproduction based on Multi-Representational Learning from Demonstrations},
  author={Brendan Hertel and S. Reza Ahmadzadeh},
  booktitle={20th International Conference on Advanced Robotics (ICAR)},
  year={2021},
  organization={IEEE}
}
```

The key steps as implemented in code are explained below:

1. Setup
   1. To initialize a SAMLfD object, call `mlfd.SAMLfD()` after importing mlfd.py
   1. to use a given demonstration, call the method `add_traj(demonstration)`. This stores the demonstration for the object to use. Note: the demonstration should be in the form of a n_pts x n_dims numpy array.
   1. To use a representation, call the method `add_representation(representation, 'Name')`, where representation is a function handle to the representation, which takes its inputs in the form of `f(org_traj, constraints, index)` and only returns the deformed trajectory in the same general shape as the demonstration. `'Name'` is the name of the representation, and is only used for labeling data shown to the user.
   1. To use a similarity metric, call the method `add_metric(metric)`, where metric is a function handle to the metric, which takes its inputs in the form of `f(org_traj, comparison_traj)` where both org_traj and comparison_traj  are n_pts x n_dims numpy arrays (for reference, metrics in the [similaritymeasures python package](https://pypi.org/project/similaritymeasures/) are set up this way). If the metric is a dissimilarity metric and not a similarity metric, then pass `True` as a second argument to this function call.
1. Create Meshgrid
   1. To create the meshgrid call the method `create_grid()`. This function has many optional inputs, such as the point density of the grid (`given_grid_size`, given as an integer, defaults to 9), distance of side lengths of similarity region (`dists`, given as an iterable object with n_dims float elements, defaults to curve length/8), generalization point (`index`, given as an integer which determines which index in the demonstration is the deform point, default is index 0), and whether or not to plot the resulting meshgrid (`plot`, given as a boolean, dafult is `False`). Note: the Meta-LfD object automatically assumes that the initial point and endpoint are constrained in the reproduction. If either the initial or endpoint is given as a generalization point, that constraint is lifted. To manually lift these constraints, set the `constrain_init` or `constrain_end` member variables to `False`.
1. Deform from Grid
   1. To get deformations from each point on the grid, call the method `deform_traj()`. This will go through each point on the grid and record each deformation to be used in similarity evaluation later. There is one optional argument to this function, `plot`. The default is `False`, but if set to `True`, will show a plot of all the deformations (only works for 1 or 2 dimensions).
1. Similarity Evaluation
   1. To calculate the similarities of each deformation call the method `calc_similarities()`. This will go through each deformation and compare it to the demonstration using the similarity metric provided. These similarity values are then stored to be interpreted by the framework later. This method has one optional argument, `downsample` (`True` by default). If `True`, trajectories are downsampled to 100 points before evaluating similarity (if trajectories are already 100 points or less, no downsampling is performed). This is done for speed. To visually see the similarity values, call the method `plot_heatmap(mode, filepath)`. This will plot the similarity values into heatmaps (only works for <=3 dimensions). `mode` is an argument that determines whether to save the plots to the directory specified by `filepath` or show the plots to the user (given by a string, eaither `'save'` (default) or `'show'`). Note: if mode is `'save'` and no filepath is given, plots are saved to the working directory.
   1. (Optional) The above calculations require significant computation. If the combination of demonstration, representations, and metric are likely to be used again, it is recommended you save your results to be loaded from later. To save results, call the method `save_to_h5(filename)`, which saves data from above steps. `filename` should be string which has the full filepath of the save file, including the save data file name which must end in `'.h5'`. To load saved data, call the method `load_from_h5(filename)` where `filename` is same as before. Note: when loading from a file, the Setup step must still be done, but meshgrid creation, grid deformation, and similarity evaluation are all taken from loaded data.
1. Interpret Similarity
   1. In order to understand the best similarity metric, the best metric at each point in the meshgrid must be found. To determine this, call the method `get_strongest_sims(threshold)`. This will find the representation with greatest similarity that is above `threshold` (a float from [0, 1], default 0.1) at each point. Note about `threshold`: it is not recommended that you choose either 0.0 or 1.0 as values for `threshold`, as there is a possibility that a single value will be passed to the classifier, in which case it will fail. To view the results of this similarity interpretation, call the method `plot_strongest_sims(mode, filepath)`. This will show each representation with a color on the meshgrid, where a point with a certain color shows the best representation at that point. See 4i for explanations of `mode` and `filepath`.
1. Set up classifier
   1. To set up the classifier, call the method `set_up_classifer()`. This will set up a support vector machine classifier (SVC) as implemented by [sklearn](https://scikit-learn.org/stable/modules/generated/sklearn.svm.SVC.html). Note: by default, the Meta-LfD object uses the default parameters of the SVM. To change parameters, it is recommended you edit this function.
1. User-Desired Outputs
   1. Viewing similarity regions. To view the similarity regions, call the method `plot_classifier_results(mode, filepath)`. This is a general function to handle <=3 dimensional outputs. For 2D, a better output can be seen by the method `plot_countour2D(mode, filepath)`. For 3D, a better output can be seen by the method `plot_cube3D(mode, filepath)`. See 4i for explanations of `mode` and `filepath`.
   1. To get the reproduction with greatest similarity at a specific point, call the method `reproduce_at_point(coords, plot)`, where `coords` is a 1 x n_dims numpy array with the coordinates of the new deformation point and `plot` is a boolean (`False` by default), which if set to `True`, will plot the reproduction alongside the original deformation. This function returns the reproduced trajectory in the same general shape as the demonstration. If you are unsure of where you want a reproduction, you can call the methods `reproduction_point_selection2D()` or `reproduction_point_selection3D()`. These will show the similarity region to the user and allow them to click on a point in that similarity region and create a reproduction. Both of these functions have the `plot` option (set to `True` by default), and return the reproduced trajectory as in `reproduce_at_point`.
 
Important notes about this repository:
1. All code was built in Python 3. Currently, the only version issue between Python 3 and Python 2 found in this repository is a single line in the [mlfd.py](https://github.com/brenhertel/Meta-LfD/blob/master/scripts/mlfd.py#L794) which should be changed from `*zip` to `zip` to work in Python 2.
1. The code uses the .h5 file format to store data, which is done through the h5py python package. This follows the HDF5 format. More information on HDF5 can be found [here](https://support.hdfgroup.org/HDF5/), and more information on h5py can be found [here](https://www.h5py.org/).
1. In the [data folder](https://github.com/brenhertel/Meta-LfD/tree/master/data) there exists 2 different datasets in .h5 files. The [LASA handwriting dataset](https://bitbucket.org/khansari/lasahandwritingdataset/src/master/), stored in the file lasa_dataset.h5, and the [Georgia Tech dataset](https://sites.google.com/view/rail-lfd/home?authuser=0), which can be found across 4 files, seperated by skill type: PRESSING_dataset.h5, PUSHING_dataset.h5, REACHING_dataset.h5, WRITING_dataset.h5.
1. Along with the implementation of Meta-LfD 3 LfD representations are implemented. They are:
   1. The Jerk-Accuracy Model (JA) - [Paper](https://www.researchgate.net/publication/305628807_Geometrical_Invariance_and_Smoothness_Maximization_for_Task-Space_Movement_Generation), [MATLAB](https://www.mathworks.com/matlabcentral/fileexchange/58403-kinematic-filtering-for-human-and-robot-trajectories), [Python](https://github.com/brenhertel/Meta-LfD/blob/master/scripts/ja.py)
   1. Laplacian Trajectory Editing - [Paper](https://link.springer.com/article/10.1007/s10514-015-9442-3), [Python](https://github.com/brenhertel/Meta-LfD/blob/master/scripts/lte.py)
   1. Dynamic Movement Primitives (DMP) - [Paper](https://www.cs.utexas.edu/users/sniekum/classes/RL-F17/papers/Pastor09.pdf), [Python](https://github.com/brenhertel/Meta-LfD/tree/master/scripts/dmp_pastor_2009)
1. For all examples shown below, JA is shown in red, LTE is shown in green, DMP is shown in blue, and the original demonstration is shown in black.
 
Some examples are provided in the repository:
1. The Meta-LfD framework using a 2D initial point deformation. This uses the 'Saeghe' shape from the [LASA handwriting dataset](https://bitbucket.org/khansari/lasahandwritingdataset/src/master/). The outputs of the framework can be seen [here](https://github.com/brenhertel/Meta-LfD/tree/master/example_outputs/2d_initpt_example) including heatmaps, similarity regions, and example reproductions. For more details on the process, see [python script](https://github.com/brenhertel/Meta-LfD/blob/master/scripts/2d_initpt_example_prcess.py).
   1. Demonstration <br/> ![Demonstration](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/2d_initpt_example/Original%20Trajectory.png)
   1. Meshgrid <br/> ![Meshgrid](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/2d_initpt_example/meshgrid.png)
   1. Deformations Grid <br/> ![Deformations Grid](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/2d_initpt_example/deformations.png)
   1. Heatmaps <br/> ![JA Heatmap](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/2d_initpt_example/JA%20Heatmap.png) ![LTE Heatmap](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/2d_initpt_example/LTE%20Heatmap.png) ![DMP Heatmap](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/2d_initpt_example/DMP%20Heatmap.png)
   1. Similarity Contour <br/> ![Similarity Contour](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/2d_initpt_example/Similarity%20Contour.png)
   1. Reproduction <br/> ![Reproduction](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/2d_initpt_example/example_reproduction.png)
 
1. The Meta-LfD framework using a 3D initial point deformation. This uses a reaching skill demonstration from the [Georgia Tech dataset](https://sites.google.com/view/rail-lfd/home?authuser=0). The outputs of the framework can be seen here: <insert fpath link> including heatmaps, similarity regions, and example reproductions. For more details on the process, see [python script](https://github.com/brenhertel/Meta-LfD/blob/master/scripts/3d_initpt_example_prcess.py).
   1. Demonstration <br/> ![Demonstration](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/3d_initpt_example/Original%20Trajectory.png)
   1. Meshgrid <br/> ![Meshgrid](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/3d_initpt_example/meshgrid.png)
   1. Heat "cubes" <br/> ![JA Heat "cubes"](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/3d_initpt_example/JA%20Heatcube.png) ![LTE Heat "cubes"](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/3d_initpt_example/LTE%20Heatcube.png) ![DMP Heat "cubes"](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/3d_initpt_example/DMP%20Heatcube.png)
   1. Similarity Cube <br/> ![Similarity Cube](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/3d_initpt_example/Similarity%20Region%20Cube.png)
   1. Reproduction <br/> ![Reproduction](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/3d_initpt_example/example_reproduction.png)
 
1. The Meta-LfD framework using a 3D endpoint deformation. This uses  a pressing skill demonstration from the [Georgia Tech dataset](https://sites.google.com/view/rail-lfd/home?authuser=0). The outputs of the framework can be seen here: <insert fpath link> including heatmaps, similarity regions, and example reproductions. For more details on the process, see [python script](https://github.com/brenhertel/Meta-LfD/blob/master/scripts/3d_endpt_example_prcess.py).
   1. Demonstration <br/> ![Demonstration](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/3d_endpt_example/Original%20Trajectory.png)
   1. Meshgrid <br/> ![Meshgrid](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/3d_endpt_example/meshgrid.png)
   1. Heat "cubes" <br/> ![JA Heat "cubes"](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/3d_endpt_example/JA%20Heatcube.png) ![LTE Heat "cubes"](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/3d_endpt_example/LTE%20Heatcube.png) ![DMP Heat "cubes"](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/3d_endpt_example/DMP%20Heatcube.png)
   1. Similarity Cube <br/> ![Similarity Cube](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/3d_endpt_example/Similarity%20Region%20Cube.png)
   1. Reproduction <br/> ![Reproduction](https://github.com/brenhertel/Meta-LfD/blob/master/example_outputs/3d_endpt_example/example_reproduction.png)
