# psf_dasnoisepy
This repository has the software and workflows for ambient-noise analysis of the PSF DAS data. From Onyx acquisition to dispersion and monitoring analysis.

The data is assumed to be copied from the Onyx system to petasaur into a specific drive.

Todo:
* modify the correlation scripts to have a DASnoise_module. DAS_module should contain utils just for the DAS.
* add an FK filter in the pre-processing module.
* rename noise_processing to noise_fft, because processing does indicates that the output data is a Fourier spectrum.
* add noisepy dv/v module into the DAS_module.

The workflow is as follow:

1. Convert H5 to the S3 object in Zarr format (example of scripts [here](https://github.com/niyiyu/NoisePy4DAS-SeaDAS/blob/main/DASStore/scripts/mpi_sermeq2pnwstore1_OOI_DAS_1d.py)). The H5 data is on petasaur, the output data should be on the minio machine (or pnwstore1). 
2. Cross correlations: this script, a modification of [S1](https://github.com/niyiyu/NoisePy4DAS-SeaDAS/blob/main/src/S1_preprocess_correlate.py) compute the cross correlations, stacked within each file. It uses noisepy module DASnoise. @Yiyu can you modify the client to be pnwstore1 (or our future minio machine). Rename the scripts to S1_cross_correlation. 
3. Stacking: this script will be a modification of this [one](https://github.com/niyiyu/NoisePy4DAS-SeaDAS/blob/main/src/S2_stacking.py), though it does not seem to have been updated with the zarr format.
4. Dispersion analysis from the stacks:
  * write a script to select the channel pairs with expected Rayleigh waves, and the channel pairs with expected Love waves. To the first order, this should be calculated following: the azimuth of the cable is given by the azimuth of a given pair of adjacent channels (N, N+1): az_chan(N)= az(chanN+1,chanN). For each channel virtual source *N*, gather all pairs that have the following condition: ``(360-tol< az_chan < tol) & (-tol+180< az_chan < tol+180(`` (``tol`` is a tolerance angle between 10 and 30), which would be pairs of Rayleigh waves, and ``(-tol+90< az_chan < tol+90) &(-tol+270< az_chan < tol+270) ``, which would be pairs of Love waves. Output these stacks into 1 Zarr file with 2 DataSets: 
    1) Rayleigh_wave: 3D data volume of dimension: lag time, distance, virtual source.
    2) Love_wave: 3D data volume of dimension: lag time, distance, virtual source.
  * write a script to make plots of these xcorr and allow for filtering parameters. This will help users determine the frequency band of their signals
  * Perform F-K analysis on continuous sub-section of the cable by moving the virtual source by one channel at a time (smoothly running subarray). User defines the width of the subsection depending on freqency bandwidth and energy loss with distance.
  * Dispersion curve extraction (???, maybe we design an ML model for it?)
  * package scripts to perform MCMC inversion of each dispersion curve into a Vs profile. An example from Brad is [here](https://github.com/bradlipovsky/whidbey-surface-waves). the VS profile will be assigned to each channel.
  * Combine the Vs profiles to form a 2D image.
  
5. Monitoring analysis:
  * Stretching for single-channel xcorr. If noisy, have the option to stack the nearby xcorr within a specific inter-chanel distance.
  * Stretching for the inter-channel xcorr. User specify a minimum velocity for picking the **coda** waves, or test using the **surface wave** peak measurements. User defines frequency band. Scripts should include a noise-floor analysis to see when to stop the coda.
  * Pull DTS data and interpolate in time and space to match the dv/v per channel and at a given time.
  * Pull weather data from weather reanalysis . Example is written by PMakus [here](https://github.com/Denolle-Lab/Mt-St-Helens/blob/main/era5_data_request.ipynb)
  * Output time series as an Xarray into Zarr.


