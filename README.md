
!pip install matplotlib

%matplotlib inline
import numpy as np
import matplotlib.pyplot as plt
from astropy.time import Time
from scipy.constants import c
from scipy.optimize import curve_fit

#%matplotlib qt

plt.rcParams['figure.figsize'] = [16, 9]
plt.rcParams['figure.facecolor'] = 'w

offset =  66
f_carrier = 2241.6e6 + offset
print(f_carrier)

offset2 =  -3641
f_carrier2 = 2268e6 + offset2
print(f_carrier2)

data = np.fromfile('/content/ch3_data.dat', sep = ' ').reshape((-1,4))
t_data = Time(data[:,0], format = 'mjd')
freq_data = data[:,1]
amp_data = data[:,2]

data2 = np.fromfile('/content/ch3_data1.dat', sep = ' ').reshape((-1,4))
t_data2 = Time(data2[:,0], format = 'mjd')
freq_data2 = data2[:,1]
amp_data2 = data2[:,2]


gmd_file = '/content/jpl_0824.dat'
gmd_mjd = []
gmd_rangerate = []
with open(gmd_file) as f:
    for l in f.readlines()[0:]:
        gmd_mjd.append(float(l.split()[0]))
        gmd_rangerate.append(float(l.split()[-1]))
gmd_mjd = np.array(gmd_mjd)
gmd_mjd = gmd_mjd - 2400000.5
gmd_rangerate = np.array(gmd_rangerate)

t_gmd = Time(gmd_mjd , scale = 'utc', format = 'mjd')


f_carrier = 2241.6e6
f_carrier2 = 2268e6
f_gmat = f_carrier * (1 - 1e3*gmd_rangerate/c)

rangerate_interp = np.interp(t_data.utc.mjd, t_gmd.utc.mjd, gmd_rangerate)
freq_gmat = f_carrier * (1 - 1e3*rangerate_interp/c)
f_gmat2 = f_carrier2 * (1 - 1e3*gmd_rangerate/c)

rangerate_interp2 = np.interp(t_data2.utc.mjd, t_gmd.utc.mjd, gmd_rangerate)
freq_gmat2 = f_carrier2 * (1 - 1e3*rangerate_interp2/c)



# Create subplots
fig, ax1 = plt.subplots()

# Plot first carrier frequency data
ax1.plot(t_data.datetime, freq_gmat, '.', markersize=20, color='tab:red', label='JPL Horizons - 2241.6MHz')
ax1.plot(t_data.datetime, freq_data, '.', markersize=4, color='tab:green', label='VE7TIL Data - 2241.6MHz')

# Set labels and colors for the first y-axis
ax1.set_xlabel('UTC datetime')
ax1.set_ylabel('Frequency (Hz)', color='tab:blue')
ax1.tick_params(axis='y', labelcolor='tab:blue')


plt.ylim(-2, 2)  # Set the y-axis limits to -2 and 2
plt.plot(t_data2.datetime, freq_data2 - freq_gmat2, '.', markersize=2, label='VE7TIL Residuals vs JPL Horizons Data')
plt.title('Chandrayaan-3 Frequency Residuals offset - 2268.0MHz\n%s to %s' % (t_data2.datetime[0].strftime('%Y-%m-%d'), t_data2.datetime[len(freq_data2)-1].strftime('%Y-%m-%d')))
plt.ylabel('Frequency (Hz)')
plt.xlabel('UTC time')
plt.legend()
plt.show()
plt.show(block=True)


plt.ylim(-3,3)
plt.plot(t_data.datetime, freq_data - freq_gmat,'.',markersize=2,  label = 'VE7TIL Residuals vs JPL Horizons Data')
plt.title('Chandrayaan-3 Frequency Residuals - 2241.6MHz\n%s to %s'%(t_data.datetime[0].strftime('%Y-%m-%d'), t_data.datetime[len(freq_data)-1].strftime('%Y-%m-%d')))
plt.ylabel('Frequency (Hz)')
plt.xlabel('UTC time')
plt.legend()
plt.show();
