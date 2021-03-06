.. _code-examples:

Introduction
------------

.. index:: pair: code; examples
           single: sourcecode

The Short Range Weather (SRW) Application is one of the many configurations of the Unified Forecasting System (`UFS <https://ufs-weather-model.readthedocs.io/en/ufs-v2.0.0/>`__), which is a community-based, coupled, comprehensive Earth modeling system. The UFS is an open-sourced project made available to the public and researchers, and is the future forecasting suite for NOAA’s operational numerical weather prediction applications.

As mentioned earlier, the UFS can be configured into `multiple applications <https://ufscommunity.org/science/aboutapps/>`__, and this document describes the SRW Application, which predicts atmospheric behavior on a limited spatial domain with time scales from less than an hour to several days. The SRW Application consists of a prognostic atmospheric model, pre and post processing, and community workflow. Future work is to include data assimilation and verification packages as part of the workflow.

This document will expound on each piece of the SRW Application as well as discuss the workflow for the containerized SRW Application.

Pre-processor Utilities and Initial Conditions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

There are a number of pre-processing utilities to initialize for model integration. For example, the regional grid, orography, and surface climatology files are required to be generated first before the limited area model. Chgres_cube, a pre-processing software, is used to convert external raw model data into initial and lateral boundary conditions, which are saved as netCDF format that are then used as input files to the forecast model. More information about the UFS pre-processor utilities can be found `here <https://noaa-emcufs-utils.readthedocs.io/en/ufs-v2.0.0/>`__.

Forecast Model
^^^^^^^^^^^^^^

The Finite-Volume Cubed-Sphere (`FV3 <https://noaa-emc.github.io/FV3_Dycore_ufs-v2.0.0/html/index.html>`__) dynamic core LAM configured is the prognostic atmospheric model in the UFS SRW Application. Currently, there are three supported model resolutions (3, 13, and 25-km) for the SRW with 64 vertical levels. The Extended Schmidt Gnomonic (ESG) grid uses FV3-LAM because it displays relatively uniform grid cells across the domain.

The Common Community Physics Package (`CCPP <https://dtcenter.org/community-code/common-community-physics-package-ccpp>`__) are packages used by the model that describe small-scale physics like clouds and radiations. There are two CCPP packages currently being worked on: the Rapid Refresh Forecast System (RRFS) and the Global Forecast System (GFS). A scientific description of the CCPP parameterizations and suites can be found `here <https://dtcenter.ucar.edu/GMTB/v5.0.0/sci_doc/index.html>`__.

Both GRIB2 and NEMSIO are acceptable input data for the SRW app. The UFS Weather Model ingests initial and lateral boundary conditions files produced by chgres_cube and output files in NetCDF format to specific projections.

Post-processor
^^^^^^^^^^^^^^

The Unified Post Processor is part of the workflow for the SRW Application, and converts the NetCDF output to GRIB2 format. These newly converted files can be used to visualize, plot, and verify the weather model output. Learn more about UPP `here <https://upp.readthedocs.io/en/upp-v9.0.0/>`__.

Visualization 
^^^^^^^^^^^^^^

A python script is used to generate simple visualizations for the SRW Application. This visualization script is part of the regional_workflow and is used to help users ensure the application is producing reasonable results. The script can output about a dozen different weather variables as well as do a comparison between two runs if both have the same domain and resolution.

Build System and Workflow
^^^^^^^^^^^^^^^^^^^^^^^^^

A CMake-based build system is used to build the required components for running the end-to-end SRW Application, which include the pre and post processing software and UFS WM but not the `NCEPLIBS-external <https://ufs-srweather-app.readthedocs.io/en/ufs-v1.0.1/Glossary.html#term-nceplibs-external>`__ and `NCEPLIBS <https://ufs-srweather-app.readthedocs.io/en/ufs-v1.0.1/Glossary.html#term-nceplibs>`__ packages. Both NCEPLIBS packages are necessary to build the UFS WM and are preinstalled on some `NOAA HPC machines <https://github.com/ufs-community/ufs-srweather-app/wiki/Supported-Platforms-and-Compilers>`__. In the case of the containerized SRW Application, the model is already built.

Once the SRW Application is built, an experiment generator script can be used to create a Rocoto-based workflow that runs each task in the system in the proper sequence. If the Rocoto software isn’t present on the platform, the components of running the SRW Application can be run individually.

Containerized SRW Application
-----------------------------

Overview
^^^^^^^^

Containerization offers a lot of advantages like ease of portability to on-prem or cloud platforms, ensuring consistent and efficient builds, and faster development cycles, just to name a few. An advantage specific to this project is that a containerized SRW Application allows users to run test cases on the model quickly and efficiently without worrying about the SRW Application failing. This should allow the users to easily learn and quickly become more comfortable with the SRW Application by removing the requirement to build the SRW Application itself.

A sample case was created for the containerized SRW Application and the instructions to run said case can be found in the following sections. It should be noted that refinement of the containerized SRW Application is anticipated as this project is still considered a proof of concept.

Container Type
^^^^^^^^^^^^^^

`Docker <https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&cad=rja&uact=8&ved=2ahUKEwicleXLoZj1AhVKnGoFHTfMAhMQFnoECAMQAQ&url=https%3A%2F%2Fwww.docker.com%2Fresources%2Fwhat-container&usg=AOvVaw010ad-skRSEl9ymUmyidiy>`__ is one of the most recognized container software companies used by many in the industry. Which is why the current SRW Application (ufs-v1.0.1) includes `documentation <https://ufs-srweather-app.readthedocs.io/en/ufs-v1.0.1/Docker.html>`__ on how to run the model in Docker. But working with Docker can provide additional security risks if not used correctly. Because of this, `Singularity <https://sylabs.io/guides/3.5/user-guide/introduction.html>`__ containers were used for this project. Singularity is a crowd-supported container software company that specializes in running containers on High Performance Computing (HPC) platforms and includes additional security measures.

Sample Case Workflow
^^^^^^^^^^^^^^^^^^^^

As mentioned earlier the containerized SRW Application is in a proof of concept state. Meaning the workflow isn’t as elegant as it could be and contains a lot of manual steps. Therefore, users should be comfortable running basic linux commands like declaring variables, copying directories, and modifying scripts. More development is expected on the workflow like using scripts to automate tasks, which should result in a more enjoyable user experience.

The current sample case can only be performed on Orion, a NOAA HPC machine, at the moment. However, it is anticipated that the containerized SRW Application will run on more platforms including on-prem and cloud platforms in the future.

Users that run the workflow will need some prerequisites in order to run the sample case. At the present time, the sample case is quite simple. As it will have users set up and run the SRW Application inside of a singularity container. Unfortunately, there are no visualizations tools like ncviewer and wgrib2 within the container but these tools are expected to be added in future releases.

With no issues, the sample case workflow can be completed in about an hour. The workflow runs the SRW Application on the RRFS_CONUS_25km grid, with initial GFS ICS and LBCS at 0.25 degree resolution (found locally on Orion) at a forecast length of 12 hours. The most recent release of the SRW Application (ufs-v1.0.1) was used in this sample case.

Prerequisites
'''''''''''''

The following items are needed to run the containerized SRW Application:

-  Access to `Orion <https://www.noaa.gov/organization/information-technology/orion>`__: Those on the EPIC project received access to Orion shortly after receiving our NOAA card/credentials. Once approved, you’ll have write access to the epic project (epic-ps), which will allow you to create your work directory (i.e. username) if it isn’t already there.
-  The SRW Application Singularity image built from `Docker <https://hub.docker.com/r/noaaepic/ubuntu20.04-epic-srwapp>`__
-  Workflow instructions (found in the next section)

Running the Sample Case Workflow
''''''''''''''''''''''''''''''''

The Sample Case Workflow has been broken down into three sections:

-  Orion and Singularity Setup

-  Work Env and Conda Setup

-  Preparing and Running the Workflow
   
Orion and Singularity Setup:

* Log into Orion using the command below with your username::

     ssh -x username@Orion-login.hpc.msstate.edu
     Note: username is first initial followed by last name, example: John Smith’s username is jsmith

* Once on the machine, run the following commands below to download the SRW Application Singularity Image from Docker and convert it to a Singularity sandbox::

     cd /work/noaa/epic-ps/$USER
     Note: if your $USER doesn’t exist, you may create it by replacing the ‘cd’ with ‘mkdir -p’ in the command above. $USER is the same username used in the previous step.
     module load singularity
     singularity pull ubuntu20.04-epic-srwapp.sif docker://noaaepic/ubuntu20.04-epic-srwapp:latest
     Note: if you run out of space downloading the docker image, you can create a symbolic link of the singularity module from your home directory to the work directory by doing the following:
           cd /home/$USER
	   mv .singularity /work/noaa/epic-ps/$USER
	   ln -s /work/noaa/epic-ps/esnyder/.singularity .
     singularity build --sandbox ubuntu20.04-epic-srwapp ubuntu20.04-epic-srwapp.sif

* After the Singularity sandbox is complete, run the following commands to obtain a Slurm job allocation from Orion so that the SRW Application can run. Please note, depending on *Orion workload status*, allocating resources for this project can take minutes to an hour to complete. More on what the salloc command does is found `here <https://slurm.schedmd.com/salloc.html>`__::

     salloc -N 1 -n 40 -A epic-ps -q batch -t 2:30:00 --partition=orion

* Run the command below to check if any resources have been assigned::

     squeue -u $USER
     Note: if resources have been assigned, the output will look something like this:
           JOBID   PARTITION NAME USER   ST TIME    NODES NODELIST(REASON)
           3500248 orion bash     jsmith R  1:22:27 1     Orion-23-03
	   
* Once resources have been allocated, ssh into the allocated compute node but note that the compute node has no access to the internet::

     ssh Orion-##-##
     Note: in the example above, the command would be: ssh Orion-23-03.
     

Work Env and Conda Setup:
                        
* After you are logged into the compute node, run the following commands to set up Singularity sandbox and work environments::

     module load singularity
     cd /work/noaa/epic-ps/$USER
     cp ~/.bashrc .
     singularity shell -e --bind /work:/work -w ./ubuntu20.04-epic-srwapp
     Note: in the example above, Orion /work directory is mounted in the container. You may need to create /work directory inside the conainer sandbox.
     source .bashrc
     Note: if you experience trouble sourcing variables feel free to use the bashrc script found in ufs-srweather-app/docs repo.
    
* Now that your Singularity and work environments are setup, let’s setup the Conda environment for Orion by running the following commands::

     source /ubuntu20.04-epic-srwapp/opt/ufs-srweather-app/env/wflow_orion_gnu.env
     source /ubuntu20.04-epic-srwapp/opt/ufs-srweather-app/env/build_orion_gnu.env
     cp -rf /ubuntu20.04-epic-srwapp/opt/bin /ubuntu20.04-epic-srwapp/opt/ufs-srweather-app
     
Preparing and Running the Workflow
                                  
* Navigate to the ush directory, by running the command below::

     cd /ubuntu20.04-epic-srwapp/opt/ufs-srweather-app/regional_workflow/ush

* Verify if the config shell script is accurate by comparing it to ``config.sh`` found in the ufs-srweather-app/docs repo. Note that the experiment directory ``EXPT_SUBDIR`` variable in the config file is where the UFS Weather Model output files will be written to. The experiment directory default value is ``community`` and can be modified by the user::

     vi config.sh
     Note: if config file is missing, simply copy the one from ufs-srweather-app/docs repo by doing the following:
           vi config.sh
	   press ‘i’
	   Copy contents of the config.sh file on ufs-srweather-app/docs repo
           right click in the terminal window (this should paste the contents of the file here).
	   Press ‘esc’
	   Type :x
	   Hit enter

* To ensure the model can run on Orion, replace the ‘srun’ command with ‘mpirun -np 12’ command for the following scripts: exregional_make_sfc_climo.sh, exregional_make_ics.sh, exregional_make_lbcs.sh, and exregional_run_fcst.sh::

     vi /ubuntu20.04-epic-srwapp/opt/ufs-srweather-app/regional_workflow/scripts/exregional_make_sfc_climo.sh
     Search for ORION by doing: /ORION
     Enter interactive mode: ‘i’ and replace ARN var from ‘srun’ to ‘mpirun -np 12’
     Save the file using ‘esc’, ‘:x’
     Repeat this process for the three remaining shell scripts

* Now you are ready to run the SRW Application workflow. The workflow has been broken down into individual scripts. More information on what each of these scripts do, can be found `here <https://ufs-srweather-app.readthedocs.io/en/ufs-v1.0.1/SRWAppOverview.html#description-of-workflow-tasks>`__. Please run the following scripts in order.
    * Fetch external data for initial conditions based on the case cycle (Estimated run time: < 1 min)::
    
	./run_get_ics.sh
	A successful execution of the script looks like this
	....
        Creating symlinks in the staging directory (extrn_mdl_staging_dir) to the
        external model files on disk (extrn_mdl_fns_on_disk) in the source directory 
        (extrn_mdl_source_dir):
        extrn_mdl_source_dir = "/work/noaa/fv3-cam/UFS_SRW_app/v1p0/model_data/FV3GFS/2019061500"
        extrn_mdl_fns_on_disk = ( "gfs.pgrb2.0p25.f000" )
        extrn_mdl_staging_dir = "path-to-expt_dirs/2019061500/FV3GFS/for_ICS"
        
        ========================================================================
        Successfully copied or linked to external model files on disk needed for
        generating initial conditions and surface fields for the FV3 forecast!!!
        
        Exiting script:  "exregional_get_extrn_mdl_files.sh"
        In directory:    "path-to-ufs-srweather-app/regional_workflow/scripts"
        ========================================================================
        
        ========================================================================
        Exiting script:  "JREGIONAL_GET_EXTRN_MDL_FILES"
        In directory:    "path-to-ufs-srweather-app/regional_workflow/jobs"
        ========================================================================
	
    * Fetch external data for lateral boundary conditions based on the case cycle. (Estimated run time: < 1 min)::
    
	./run_get_lbcs.sh
	A successful execution of the script looks like this
	....
	Creating symlinks in the staging directory (extrn_mdl_staging_dir) to the
	external model files on disk (extrn_mdl_fns_on_disk) in the source directory 
	(extrn_mdl_source_dir):
	extrn_mdl_source_dir = "/work/noaa/fv3-cam/UFS_SRW_app/v1p0/model_data/FV3GFS/2019061500"
	extrn_mdl_fns_on_disk = ( "gfs.pgrb2.0p25.f006" "gfs.pgrb2.0p25.f012" "gfs.pgrb2.0p25.f018" "gfs.pgrb2.0p25.f024" "gfs.pgrb2.0p25.f030" "gfs.pgrb2.0p25.f036" "gfs.pgrb2.0p25.f042" "gfs.pgrb2.0p25.f048" )
	extrn_mdl_staging_dir = "path-to-expt_dirs/test_GST/2019061500/FV3GFS/for_LBCS"
	
	========================================================================
	Successfully copied or linked to external model files on disk needed for
	generating lateral boundary conditions for the FV3 forecast!!!
	
	Exiting script:  "exregional_get_extrn_mdl_files.sh"
	In directory:    "path-to-ufs-srweather-app/regional_workflow/scripts"
	========================================================================
	
	========================================================================
	Exiting script:  "JREGIONAL_GET_EXTRN_MDL_FILES"
	In directory:    "path-to-ufs-srweather-app/regional_workflow/jobs"
	========================================================================
	
    * Pre-processing task to generate regional grid files (Estimated run time: < 1 min).::
    
	./run_make_grid.sh
	A successful execution of the script looks like this
	....
	path-to-ufs-srweather-app/regional_workflow/ush/set_namelist.py -q -n /work/noaa/epic-ps/jongkim/temp/expt_dirs/test_GST/input.nml.base -u ''\''namsfc'\'': {
	'\''FNALBC'\'': ../fix_lam/C403.snowfree_albedo.tileX.nc,
	'\''FNALBC2'\'': ../fix_lam/C403.facsf.tileX.nc,
	'\''FNTG3C'\'': ../fix_lam/C403.substrate_temperature.tileX.nc,
	'\''FNVEGC'\'': ../fix_lam/C403.vegetation_greenness.tileX.nc,
	'\''FNVETC'\'': ../fix_lam/C403.vegetation_type.tileX.nc,
	'\''FNSOTC'\'': ../fix_lam/C403.soil_type.tileX.nc,
	'\''FNVMNC'\'': ../fix_lam/C403.vegetation_greenness.tileX.nc,
	'\''FNVMXC'\'': ../fix_lam/C403.vegetation_greenness.tileX.nc,
	'\''FNSLPC'\'': ../fix_lam/C403.slope_type.tileX.nc,
	'\''FNABSC'\'': ../fix_lam/C403.maximum_snow_albedo.tileX.nc,
	}' -o path-to-expt_dirs/test_GST/input.nml
	rm_vrfy path-to-expt_dirs/test_GST/input.nml.base

	========================================================================
	Grid files with various halo widths generated successfully!!!
	
	Exiting script:  "exregional_make_grid.sh"
	In directory:    "path-to-ufs-srweather-app/regional_workflow/scripts"
	========================================================================
	
	========================================================================
	Exiting script:  "JREGIONAL_MAKE_GRID"
	In directory:    "path-to-ufs-srweather-app/regional_workflow/jobs"
	========================================================================
    * Pre-processing task to generate regional grid files (Estimated run time: < 1 min).::
    
	./run_make_grid.sh
	A successful execution of the script looks like this
	....
	Message from "cd_vrfy" function's "cd" operation:
	path-to-expt_dirs/test_GST
	
	========================================================================
	Orography files with various halo widths generated successfully!!!
	
	Exiting script:  "exregional_make_orog.sh"
	In directory:    "path-to-ufs-srweather-app/regional_workflow/scripts"
	========================================================================
	
	========================================================================
	Exiting script:  "JREGIONAL_MAKE_OROG"
	In directory:    "path-to-ufs-srweather-app/regional_workflow/jobs"
	========================================================================
    * Pre-processing task to generate surface climatology files (Estimated run time: < 1 min).::
    
	./run_make_sfc_climo.sh
	A successful execution of the script looks like this
	....
	Message from "cd_vrfy" function's "cd" operation:
	path-to-expt_dirs/test_GST/sfc_climo/tmp
	
	========================================================================
	All surface climatology files generated successfully!!!
	
	Exiting script:  "exregional_make_sfc_climo.sh"
	In directory:    "path-to-ufs-srweather-app/regional_workflow/scripts"
	========================================================================
	
	========================================================================
	Exiting script:  "JREGIONAL_MAKE_SFC_CLIMO"
	In directory:    "path-to-ufs-srweather-app/regional_workflow/jobs"
	========================================================================
    * Generate lateral boundary conditions from external dataset (Estimated run time: < 1 min).::
    
	./run_make_lbcs.sh
	A successful execution of the script looks like this
	....
	========================================================================
	Lateral boundary condition (LBC) files (in NetCDF format) generated suc-
	cessfully for all LBC update hours (except hour zero)!!!
	
	Exiting script:  "exregional_make_lbcs.sh"
	In directory:    "path-to-ufs-srweather-app/regional_workflow/scripts"
	========================================================================
	
	========================================================================
	Exiting script:  "JREGIONAL_MAKE_LBCS"
	In directory:    "path-to-ufs-srweather-app/regional_workflow/jobs"
	========================================================================
    * Run forecast (Estimated run time: 15-20 mins).::
    
	./run_fcst.sh
	A successful execution of the script looks like this
	....
	ENDING DATE-TIME    JAN 10,2022  09:56:22.811   10  MON   2459590
	PROGRAM nems      HAS ENDED.
	* . * . * . * . * . * . * . * . * . * . * . * . * . * . * . * . * . * . * . * . 
	  *****************RESOURCE STATISTICS*******************************
	  The total amount of wall time                        = 1664.915818
	  The total amount of time in user mode                = 3272.560867
	  The total amount of time in sys mode                 = 34.391806
	  The maximum resident set size (KB)                   = 511716
	  Number of page faults without I/O activity           = 726337
	  Number of page faults with I/O activity              = 1
	  Number of times filesystem performed INPUT           = 1088
	  Number of times filesystem performed OUTPUT          = 611360
	  Number of Voluntary Context Switches                 = 942
	  Number of InVoluntary Context Switches               = 2645
	  *****************END OF RESOURCE STATISTICS*************************
	  
	  
	  ========================================================================
	  FV3 forecast completed successfully!!!

	  Exiting script:  "exregional_run_fcst.sh"
	  In directory:    "path-to-ufs-srweather-app/regional_workflow/scripts"
	  ========================================================================
	  
	  ========================================================================
	  Exiting script:  "JREGIONAL_RUN_FCST"
	  In directory:    "path-to-ufs-srweather-app/regional_workflow/jobs"
	  ========================================================================
    * Run the post-processing tool (UPP) (Estimated run time: ~5 mins).::
    
	./run_post.sh
	A successful execution of the script looks like this
	....
	========================================================================
	Post-processing for forecast hour 000 completed successfully.

	Exiting script:  "exregional_run_post.sh"
	In directory:    "path-to-ufs-srweather-app/regional_workflow/scripts"
	========================================================================
	
	========================================================================
	Exiting script:  "JREGIONAL_RUN_POST"
	In directory:    "path-to-ufs-srweather-app/regional_workflow/jobs"
	========================================================================
	
* The SRW Application creates {domain}.t{cyc}z.bgrd3df{fhr}.tmXX.grib2 files which can be found here path-to-expt_dirs/EXPT_SUBDIR/YYYYMMDDHH/postprd. Example output files are like this.::
    
    Orion-login-2[78] $ ls rrfs.*
    rrfs.t00z.natlevf000.tm00.grib2  rrfs.t00z.natlevf017.tm00.grib2  rrfs.t00z.prslevf009.tm00.grib2
    rrfs.t00z.natlevf001.tm00.grib2  rrfs.t00z.natlevf018.tm00.grib2  rrfs.t00z.prslevf010.tm00.grib2
    rrfs.t00z.natlevf002.tm00.grib2  rrfs.t00z.natlevf019.tm00.grib2  rrfs.t00z.prslevf011.tm00.grib2
    rrfs.t00z.natlevf003.tm00.grib2  rrfs.t00z.natlevf020.tm00.grib2  rrfs.t00z.prslevf012.tm00.grib2
    rrfs.t00z.natlevf004.tm00.grib2  rrfs.t00z.natlevf021.tm00.grib2  rrfs.t00z.prslevf013.tm00.grib2
    rrfs.t00z.natlevf005.tm00.grib2  rrfs.t00z.natlevf022.tm00.grib2  rrfs.t00z.prslevf014.tm00.grib2
    rrfs.t00z.natlevf006.tm00.grib2  rrfs.t00z.natlevf023.tm00.grib2  rrfs.t00z.prslevf015.tm00.grib2
    rrfs.t00z.natlevf007.tm00.grib2  rrfs.t00z.natlevf024.tm00.grib2  rrfs.t00z.prslevf016.tm00.grib2
    rrfs.t00z.natlevf008.tm00.grib2  rrfs.t00z.prslevf000.tm00.grib2  rrfs.t00z.prslevf017.tm00.grib2
    rrfs.t00z.natlevf009.tm00.grib2  rrfs.t00z.prslevf001.tm00.grib2  rrfs.t00z.prslevf018.tm00.grib2
    rrfs.t00z.natlevf010.tm00.grib2  rrfs.t00z.prslevf002.tm00.grib2  rrfs.t00z.prslevf019.tm00.grib2
    rrfs.t00z.natlevf011.tm00.grib2  rrfs.t00z.prslevf003.tm00.grib2  rrfs.t00z.prslevf020.tm00.grib2
    rrfs.t00z.natlevf012.tm00.grib2  rrfs.t00z.prslevf004.tm00.grib2  rrfs.t00z.prslevf021.tm00.grib2
    rrfs.t00z.natlevf013.tm00.grib2  rrfs.t00z.prslevf005.tm00.grib2  rrfs.t00z.prslevf022.tm00.grib2
    rrfs.t00z.natlevf014.tm00.grib2  rrfs.t00z.prslevf006.tm00.grib2  rrfs.t00z.prslevf023.tm00.grib2
    rrfs.t00z.natlevf015.tm00.grib2  rrfs.t00z.prslevf007.tm00.grib2  rrfs.t00z.prslevf024.tm00.grib2
    rrfs.t00z.natlevf016.tm00.grib2  rrfs.t00z.prslevf008.tm00.grib2
