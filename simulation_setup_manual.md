# Simulation Setup Manual

## 1. Introduction
This manual outlines the procedures for setting up, configuring, and executing a simulation run using the SKRIPS v2.0.1 environment. Following these steps ensures repeatability and consistency in data generation. This manual is written for ACC server and only for the internal use in Yonsei ASML. Please contact Ajin Cho (ajin05@yonsei.ac.kr or ajin.cho@mines.edu) if you have any questions following these instructions.

## 2. Prerequisites
Firstly, install model components and test coupled code case following the user guide of SKRIPS v2.0.1 (https://skrips.readthedocs.io/en/v2.0.1/).
To use in ACC server, I have these configurations in the ~/.bashrc_intel.skrips2.0.1 file.
<details>
<summary>.bashrc_intel.skrips2.0.1 </summary>

```
module load intel18/compiler-18
module load intel18/netcdf-4.6.1
module load intel18/openmpi-3.1.6

# Location of the modules
export SKRIPS_DIR=/home/ajin05/SKRIPS_intel/scripps_kaust_model-2.0.1/
export ESMF_DIR=$SKRIPS_DIR/esmf/
export MITGCM_DIR=$SKRIPS_DIR/MITgcm_c68r/
export WRF_DIR=$SKRIPS_DIR/WRFV452_AO/
export WRF_SRC_ROOT_DIR=$SKRIPS_DIR/WRFV452_AO/
export WPS_DIR=$SKRIPS_DIR/WPSV45/
#export PWRF_DIR=$SKRIPS_DIR/PWRFV452_AO/
export WW3_DIR=$SKRIPS_DIR/ww3_607/

export WRF_CHEM=0
# This value is 1 when installing WRF with ESMF
export WRF_ESMF=1

# Other libraries
export ZDIR=/usr/local/zlib/1.2.7
export H5DIR=/usr/local/hdf5/1.10.5_intel18
export NCDIR=/usr/local/netcdf/4.6.1_intel18
export NFDIR=/usr/local/netcdf/4.6.1_intel18
export OPENMPI=/usr/local/mpi/intel18/openmpi-3.1.6
export CMAKEDIR=/home/ajin05/anaconda3/lib/cmake

export NETCDF=$NFDIR
export NETCDF_classic=1
export LD_LIBRARY_PATH=$ZDIR/lib:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=$H5DIR/lib:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=$NCDIR/lib:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=$NFDIR/lib:$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=$OPENMPI/lib:$LD_LIBRARY_PATH

export PATH=$NCDIR/bin:$NFDIR/bin:$OPENMPI/bin:$CMAKEDIR/bin:$WPS_DIR:$PATH
export PROJ_LIB=/home/ajin05/anaconda3/share/proj/

# For ESMF
export ESMF_OS=Linux
export ESMF_COMM=openmpi
export ESMF_NETCDF=nc-config
export ESMF_OPENMP=OFF
export ESMF_LAPACK=internal
export ESMF_BOPT=g
export ESMF_ABI=64
export ESMF_COMPILER=intel
export ESMF_SITE=default
export ESMF_PIO=internal

export ESMF_LIB=$ESMF_DIR/lib/lib$ESMF_BOPT/$ESMF_OS.$ESMF_COMPILER.$ESMF_ABI.$ESMF_COMM.default/
export ESMF_MOD=$ESMF_DIR/mod/mod$ESMF_BOPT/$ESMF_OS.$ESMF_COMPILER.$ESMF_ABI.$ESMF_COMM.default/
export ESMFMKFILE=$ESMF_LIB/esmf.mk
export ESMF_TESTEXHAUSTIVE=ON
export ESMF_TESTMPMD=OFF
export ESMF_TESTHARNESS_ARRAY=RUN_ESMF_TestHarnessArray_default
export ESMF_TESTHARNESS_FIELD=RUN_ESMF_TestHarnessField_default
export ESMF_TESTWITHTHREADS=OFF

# For coupled model
export SKRIPS_NETCDF_INCLUDE="-I${NCDIR}/include -I${NFDIR}/include"
export SKRIPS_NETCDF_LIB="-L${NCDIR}/lib -L${NFDIR}/lib"
export SKRIPS_MPI_DIR=$OPENMPI
export SKRIPS_MPI_INC=$OPENMPI/include/
export SKRIPS_MPI_LIB=$OPENMPI/lib/

export LD_LIBRARY_PATH=$ESMF_LIB/:$LD_LIBRARY_PATH

# For WW3
export PATH=$WW3_DIR/model/bin:${PATH}
export PATH=$WW3_DIR/model/exe:${PATH}
export WWATCH3_ENV=$WW3_DIR/wwatch3.env
export WWATCH3_NETCDF=NC4
export NETCDF_CONFIG=${NCDIR}/bin/nc-config
# export NETCDF_CONFIG_C=${NFDIR}/bin/ncxx4-config
export NETCDF_CONFIG_F=${NFDIR}/bin/nf-config

# For DART
#export DART_DIR=/data/SO3/rus043/wrf_dart/DART-11.5.0/

echo "SKRIPS_DIR is: $SKRIPS_DIR"
```
</details>

## 3. Compiling a new case
Make new directories for wave-uncoupled case and wave-coupled case.

### Wave-uncoupled case
```
cd $SKRIPS_DIR/coupler
cp -r L3.C1.coupled_RS2012_ring ./L3.C1.coupled_nwp
```


```
cd ./L3.C1.coupled_nwp/
mv mitSettingRS mitSettingNWP


# modify utils/mitgcm_optfile.ifort
FC=mpifort
CC=mpicc
F90C=mpifort

# modify mkmod.sh
set comp      = $SKRIPS_MPI_DIR/bin/mpif77
set cccommand = $SKRIPS_MPI_DIR/bin/mpicc

# modify install.sh
cp mitSettingNWP/* code/

./install.sh
```
### Wave-coupled case

```
cd $SKRIPS_DIR/coupler
cp -r L4.C1.coupled_RS2012_ring ./L4.C1.coupled_nwp
```
The updated files for L4.C1.coupled_RS2012_ring can be downloaded from here: https://github.com/iurnus/scripps_kaust_model/tree/master/coupler/L4.C1.coupled_RS2012_ring

Compile this case using install.sh referring /data5/SKRIPS-2.0.1/L4.C1.coupled_nwp

After successful compilation, you have esmf_application in coupledCode directory. If something went wrong, I recommend you to manually follow every steps in install.sh to find out the problem. 

## 4. Preparing for initial and boundary conditions
### WPS
Download ERA5 or any other reanalysis data you need. 
<details>
<summary>getData.sh</summary>

```
#!/bin/bash -l

CODEDIR=/home/ajin05/research/model/WPS-3.9.1/ERA5/2016
DATADIR=/home/ajin05/research/model/WPS-3.9.1/ERA5/2016

# Set your python environment
# source activate root
cd $CODEDIR

DATE1=20160501
DATE2=20170301
YY1=`echo $DATE1 | cut -c1-4`
MM1=`echo $DATE1 | cut -c5-6`
DD1=`echo $DATE1 | cut -c7-8`
YY2=`echo $DATE2 | cut -c1-4`
MM2=`echo $DATE2 | cut -c5-6`
DD2=`echo $DATE2 | cut -c7-8`
Nort=60
West=110
Sout=-5
East=170

sed -e "s/DATE1/${DATE1}/g;s/DATE2/${DATE2}/g;s/Nort/${Nort}/g;s/West/${West}/g;s/Sout/${Sout}/g;s/East/${East}/g;" GetERA5-sl.py > GetERA5-${DATE1}-${DATE2}-sl.py

python GetERA5-${DATE1}-${DATE2}-sl.py

sed -e "s/DATE1/${DATE1}/g;s/DATE2/${DATE2}/g;s/Nort/${Nort}/g;s/West/${West}/g;s/Sout/${Sout}/g;s/East/${East}/g;" GetERA5-pl.py > GetERA5-${DATE1}-${DATE2}-pl.py

python GetERA5-${DATE1}-${DATE2}-pl.py

mkdir -p ${DATADIR}/$YY1

mv ERA5-${DATE1}-${DATE2}-sl.grib ERA5-${DATE1}-${DATE2}-pl.grib ${DATADIR}/$YY1/


exit 0
```
</details>

Do the works you need to generate met_em files using WPS following WPS manual such as [GitHub](https://github.com/wrf-model/WPS/blob/master/README) and [NCAR tutorial](https://www2.mmm.ucar.edu/wrf/wrf_tutorial/single_domain.html). 
* geogrid.exe
* ungrib.exe
* metgrid.exe

Here's my namelist.wps
<details>
<summary>namelist.wps </summary>

```
&share
 wrf_core = 'ARW',
 max_dom = 1,
 start_date = '2016-03-01_00:00:00','2006-08-16_12:00:00',
 end_date   = '2016-05-01_00:00:00','2006-08-16_12:00:00',
 interval_seconds = 21600
 io_form_geogrid = 2,
/

&geogrid
 parent_id         =   1,   1,
 parent_grid_ratio =   1,   3,
 i_parent_start    =   1,  31,
 j_parent_start    =   1,  17,
 e_we              = 401, 112,
 e_sn              = 513,  97,
 !
 !!!!!!!!!!!!!!!!!!!!!!!!!!!! IMPORTANT NOTE !!!!!!!!!!!!!!!!!!!!!!!!!!!!
 ! The default datasets used to produce the HGT_M, GREENFRAC,
 ! and LU_INDEX/LANDUSEF fields have changed in WPS v3.8. The HGT_M field
 ! is now interpolated from 30-arc-second USGS GMTED2010, the GREENFRAC
 ! field is interpolated from MODIS FPAR, and the LU_INDEX/LANDUSEF fields
 ! are interpolated from 21-class MODIS.
 !
 ! To match the output given by the default namelist.wps in WPS v3.7.1,
 ! the following setting for geog_data_res may be used:
 !
 ! geog_data_res = 'gtopo_10m+usgs_10m+nesdis_greenfrac+10m','gtopo_2m+usgs_2m+nesdis_greenfrac+2m',
 !
 !!!!!!!!!!!!!!!!!!!!!!!!!!!! IMPORTANT NOTE !!!!!!!!!!!!!!!!!!!!!!!!!!!!
 !
 geog_data_res = 'default','default',
 dx = 0.1,
 dy = 0.1,
 map_proj = 'lat-lon',
 ref_lat   =  25.6,
 ref_lon   = 137,
 pole_lat  = 90,
 pole_lon  = 180.
 stand_lon = -137,
 geog_data_path = '/home/share/WRF/geog/'
/
&ungrib
 out_format = 'WPS',
 prefix = 'FILE',

/

&metgrid
 fg_name = 'FILE',
 io_form_metgrid = 2,
/
```
</details>

### caseInput
Prepare MITgcm input and boundary files.

### download_wind
Generate ww3_wind.nc and ww3_current_zero.nc files by running download_wind.
```
cd $SKRIPS_DIR/coupler/L4.C1.coupled_nwp/download_wind
./download.sh
```

<details>
<summary>download.sh</summary>
#!/bin/bash -l

DATE1=20160903
DATE2=20160914
YY1=`echo $DATE1 | cut -c1-4`
MM1=`echo $DATE1 | cut -c5-6`
DD1=`echo $DATE1 | cut -c7-8`
YY2=`echo $DATE2 | cut -c1-4`
MM2=`echo $DATE2 | cut -c5-6`
DD2=`echo $DATE2 | cut -c7-8`
Nort=60
Sout=-5
West=110
East=170
#
sed -e "s/DATE1/${DATE1}/g;s/DATE2/${DATE2}/g;s/Nort/${Nort}/g;s/West/${West}/g;s/Sout/${Sout}/g;s/East/${East}/g;" getERA5-sl.py > getERA5-${DATE1}-${DATE2}-sl.py
source ~/anaconda3.sh
python ./getERA5-${DATE1}-${DATE2}-sl.py

sed -e "s/DATE1/${DATE1}/g;s/DATE2/${DATE2}/g;" processERA5.mod.py > processERA5_new.py
python processERA5_new.py

exit 0
</details>

<details>
<summary>processERA5.mod.py</summary>
    ```python
    import numpy as np
    from netCDF4 import Dataset
    import os

    # 1. 파일 설정
    input_file = 'ERA5-DATE1-DATE2-wind.nc'
    wind_out = 'ww3_wind.nc'
    curr_out = 'ww3_current_zero.nc'

    # 기존 파일 삭제 (중복 생성 에러 방지)
    for f in [wind_out, curr_out]:
        if os.path.exists(f): os.remove(f)

    # 2. 원본 데이터 읽기
    src = Dataset(input_file, 'r')
    lat_src = src.variables['latitude'][:]
    lon_src = src.variables['longitude'][:]
    time_src = src.variables['valid_time'][:]
    u10_src = src.variables['u10'][:]
    v10_src = src.variables['v10'][:]

    # 데이터 처리 (위도 반전 및 시간 단위 변환)
    lat_new = lat_src[::-1]
    u10_new = u10_src[:, ::-1, :]
    v10_new = v10_src[:, ::-1, :]
    time_new = (time_src / 3600.0) + (25567 * 24.0) # hours since 1900

    # 3. 함수: NetCDF 파일 생성 로직 (바람/해류 공용)
    def create_ww3_nc(filename, var_names, data_list):
        dst = Dataset(filename, 'w', format='NETCDF4')
        dst.createDimension('time', None)
        dst.createDimension('latitude', len(lat_new))
        dst.createDimension('longitude', len(lon_src))

        # 좌표 변수 생성
        times = dst.createVariable('time', 'i4', ('time',))
        times.units = "hours since 1900-01-01 00:00:00.0"
        lats = dst.createVariable('latitude', 'f4', ('latitude',))
        lons = dst.createVariable('longitude', 'f4', ('longitude',))

        lons[:] = lon_src
        lats[:] = lat_new
        times[:] = time_new.astype('int32')

        # 데이터 변수 생성 (u, v)
        for name, data in zip(var_names, data_list):
            v = dst.createVariable(name, 'f4', ('time', 'latitude', 'longitude'), fill_value=-32767.0)
            v.units = "m s**-1"
            v[:] = data

        dst.close()

    # 4. 파일 생성 실행
    # (1) 바람 파일 생성
    create_ww3_nc(wind_out, ['u10', 'v10'], [u10_new, v10_new])
    print("Created wind file: " + wind_out)

    # (2) 제로 해류 파일 생성 (데이터에 0.0 곱하기)
    create_ww3_nc(curr_out, ['uoce', 'voce'], [u10_new * 0.0, v10_new * 0.0])
    print("Created zero current file: " + curr_out)

    src.close()
    ```
</details>

### runCase.init - wave-uncoupeld case
### runCase.init - wave-coupled case
Modify namelist.input files as we need. After copying this new directory(/data5/SKRIPS-2.0.1/L4.C1.coupled_nwp/runCase.init.org), you can use namelist.input.real.mod and namelist.input.ini.mod files by setting the date in Allrun.sh. 

Link met_em files from your WPS directory. 
Link oceanic initial condition files from caseInput directory or pickup file from the previous run directory. 
Use modified updateLowinp.py, updateHFlux.py, and updateWave.py

Update these files: ww3_grid.inp, ww3_strt.inp, ww3_prnc.inp.wind, ww3_prnc.inp.current, ww3_shel.inp.ini, ww3_ounf.inp 
```
./Allrun
```


## 5. Setting up the new case
### runCase - wave-uncoupled case
Please check /data5/SKRIPS-2.0.1/L3.C1.coupled_nwp.ust/runCase.0910
### runCase - wave-coupled case
Please check /data5/SKRIPS-2.0.1/L4.C1.coupled_nwp/runCase

## 6. Running the model
```
qsub run_skrips.pbs
```
<details>
<summary>run_skrips.pbs</summary>
    ```bash
    #!/bin/bash

    #PBS -l nodes=3:ppn=27,walltime=48:00:00
    #PBS -q batch
    #PBS -V
    #PBS -N SKRIPS_L4.C1

    ulimit -s unlimited

    JOBID=$PBS_JOBID
    NCPU=80

    source /etc/profile.d/modules.sh
    source /home/ajin05/.bashrc_intel.skrips2.0.1

    #-- Fundamental run information
    runD="/data5/SKRIPS-2.0.1/L4.C1.coupled_nwp/runCase/"
    cd $runD

    echo "running coupled WRF--WW3--MITgcm simulation.."
    EXE="mpirun -machinefile $PBS_NODEFILE -v -np $NCPU esmf_application &> log.esmf"

    echo " "
    echo "run command: $EXE"
    eval $EXE

    echo "Done without any issue. Run ww3_ounf"; exit;
    ```
</details>

## 7. Using Cartesian coordinate
Please check directories for the L4.C4.coupled_y_mirae case. 
* /home/ajin05/research/model/WPS-3.9.1/y_mirae
* /data5/SKRIPS-2.0.1/L4.C4.coupled_y_mirae
### runCase.init
### runCase

## 8. Things left to do
Using useful packages in MITgcm: sea ice, diagnostics, etc. 
Making WRF and MITgcm to have the same wind stress. 
I tried to do this in L3.C1.coupled_nwp.ust and L4.C1.coupled_nwp.ust but failed. I hope someone could figure out the problem and solve it in the near future.

For the case I was working on, I used the updated version for these files: 
| directory | files |
|---|---|
| mitCode |exf_wind.F, ini_parms.F, kpp_routines.F, mom_fluxform.F, PARAMS.h, gad_advection.F, kpp_calc.F, mom_diagnostics_init.F, mom_vecinv.F, set_defaults.F|
| coupledCode |Makefile, mitgcm_wrf.F90, mod_esmf_atm.F90, mod_esmf_ocn.F90, mod_esmf_wav.F90, mod_types.F90, wrflib.mk|

I didn't use ECCO_CPPOPTIONS.h
'''
rm -rf mitSettingNWP/ECCO_CPPOPTIONS.h
'''

These files will be modified for our new case:

| directory | files |
|---|---|
| mitCode |CPP_OPTIONS.h, DIAGNOSTICS_SIZE.h, get_field_parameters.F, packages.conf |
| mitSetting | OBCS_OPTIONS.h, SIZE.h, exf_getffields.F, EXF_OPTIONS.h |
| coupledCode | Makefile, mod_types.F90, mod_esmf_ocn.F90 |
| utils | mitgcm_optfile.ifort, mkmod.sh |


These files will be added:
| directory | files |
|---|---|
| mitCode | seaice_advection.F, seaice_diagnostics_init.F, seaice_diffusion.F, SEAICE_OPTIONS.h, diagnostics_write.F, exf_radiation.F |