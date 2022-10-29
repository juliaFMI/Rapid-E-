'''
Created on Aug 14, 2022

    ... based on the Jupyter "Tutorial 1 - Low level data access" (D.Kiselev, 10Aug, 2022) 
    
@author: palamarc
'''
import os, sys, time
import h5py
import numpy as np
import plotly.graph_objs as go
import plotly.express as px
import datetime
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

start_time = time.time()

def anyDateToUTCDatetime(any_value):
    if isinstance(any_value, str) and len(any_value.split(':')) == 2:
        _dt = datetime.datetime.strptime(any_value, '%Y-%m-%d %H:%M').replace(tzinfo=datetime.timezone.utc)
    elif isinstance(any_value, str) and len(any_value.split(':')) == 3:
        _dt = datetime.datetime.strptime(any_value, '%Y-%m-%d %H:%M:%S').replace(tzinfo=datetime.timezone.utc)
    elif isinstance(any_value, float) or isinstance(any_value, int):
        _dt = datetime.datetime.fromtimestamp(any_value, tz=datetime.timezone.utc)
    elif isinstance(any_value, datetime.datetime):
        _dt = any_value.replace(tzinfo=datetime.timezone.utc)
    elif isinstance(any_value, pd._libs.tslibs.period.Period):
        _dt = datetime.datetime.strptime(str(any_value), '%Y-%m-%d %H:%M').replace(tzinfo=datetime.timezone.utc)
    elif isinstance(any_value, np.datetime64):
        _dt = datetime.datetime.utcfromtimestamp(
            (any_value - np.datetime64('1970-01-01T00:00:00Z')) / np.timedelta64(1, 's')
        )
    elif isinstance(any_value, list):
        _dt = []
        for _i in range(len(any_value)):
            _dt.append(anyDateToUTCDatetime(any_value[_i]))
    else:
        raise ValueError
    _ret = _dt
    return _ret


### ... Obtain your serial and device type...

#SERIAL = os.popen('hostname').read().strip().split('-')[-1]
#DEVICE = '-'.join(os.popen('hostname').read().strip().split('-')[0:-1])
#PATH = '/home/storage'

dirOutTest = 'e:/PLAIR/Rapid-Ep-00C59ACA_2021/_raw_data_proc'

SERIAL = '00C59ACA' # FMI Rapid-E+ upgraded device
DEVICE = 'Rapid-Ep'
### ... currently data are stored on "bulk": /arch/silam/bulk/data/...
PATH = 's:/data/measurements_in_situ/FMI-pollen/Plair_plus_2022/user-storage/storage'

print('Start...')
print('Your device is %s and serial is %s' % (DEVICE, SERIAL))
print('Your data is located here: \n %s' % PATH)


###############################
################# ... JP read of timestamps to be able to select the calibration sub-sets
############################ 
with open(dirOutTest+'/timeStamps_UTC_in_files_.txt', 'w') as fout:
    fout.write('file location     first_timestamp      last_timestamp \n')
    for root, dirs, files in os.walk('%s/%s-%s/data_vars' % (PATH, DEVICE, SERIAL)):  # (dirPath, dirNams, fileNams)
        for file in sorted(files):
            sample_file_path = os.path.join(root,file)  # read every file
            #print(sample_file_path)
            fout.write(sample_file_path.split('data_vars')[1])
            with h5py.File(sample_file_path, 'r') as f:
                machine_ts = np.asarray(f['time'])  # reads time stamps of all measurements within current file
                human_ts = list(map(anyDateToUTCDatetime, machine_ts))
                fout.write('        %s    %s \n'%(str(human_ts[0])[:16], str(human_ts[-1])[:16]))

print('Done!')

##################### ... JP... some tips for the command line check during debugging
### ... single line check of the open hdf5-file(f) ATTRIBUTES
####                [print('Attribute: %s = %s'%(aa,f.attrs[aa])) for aa in list(f.attrs)]
### ... single line check of the open hdf5-file(f) VARIABLES
####                 [print('Data: %s = %s'%(dat,f[dat])) for dat in sorted(f.keys())]
####-----------------------------------------------------------------------------------


elapsed_time = time.time() - start_time
print ('Computational time : ', time.strftime("%H:%M:%S", time.gmtime(elapsed_time)))
