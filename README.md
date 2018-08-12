# noaa-apt

Work in progress decoder for NOAA APT images from a recorded WAV file.

Written in Rust, never tried to do signal processing or to use Rust before...

## Alternatives

Just bought an RTL-SDR and tried to receive NOAA APT images, I'm new to this but
as of July 2018:

- [wxtoimg], by far the most popular, lots of features but the site looks dead
  forever, you can still get some binaries uploaded by some people if you are
  lucky.

- [atp-dec/apt-dec], works really good. Keep in mind that the [1.7 release]
  looks newer than the [repo's master branch]. I tried several times to compile
  the [repo's master branch] without success, later I realized that there was a
  newer [1.7 release] and it worked.

- [zacstewart/apt-decoder], written in Python, slower than the others but really
  simple. Doesn't align the image to the sync stripes.

- [martinber/apt-decoder], bad hack made by me on top of
  [zacstewart/apt-decoder] trying to align the image to the sync stripes. Still
  slow and minor artifacts on the image if you look at the vertical stripes.

[wxtoimg]: http://wxtoimg.com/
[atp-dec/apt-dec]: https://github.com/csete/aptdec
[1.7 release]: https://github.com/csete/aptdec/releases
[repo's master branch]: https://github.com/csete/aptdec
[zacstewart/apt-decoder]: https://github.com/zacstewart/apt-decoder
[martinber/apt-decoder]: https://github.com/martinber/apt-decoder

## Dependencies

- GNU Scientific Library:

  - `sudo apt install libgsl0-dev`.

- GNUPlot:

  - TODO


## Algorithm

AM resampling and demodulation using FIR filter, following method 4 or 5 in
reference [1]:

- Load samples from WAV.
- Resample and get [analytical signal].
  - Get L (interpolation factor) and M (decimation factor).
  - Get filter from a common sample rate or generate a new one:
    - Calculate impulse response of hilbert filter and lowpass filter.
    - Calculate kaiser window from parameters or use a predefined one.
    - Multiply window with impulse response for both filters.
  - Filter with lowpass only the output samples.
  - Decimate removing M-1 samples.
  - Get [analytic sygnal]:
    - Filter the signal with a hilbert filter..
    - Add the original signal, with the same delay as the filtered one.
- Get absolute value of analytic signal to finish AM demodulation.


## Analytical signal

For AM demodulation we use the [analytic signal].

### Hilbert filter

Frequency response: j for w < 0 and -j for w > 0.

Impulse response: `1/(pi*n) * (1-cos(pi*n))`

For n=0, should be 0.

## Lowpass filter

Impulse response: `sin(n*wc)/(n*pi)`.

## Notes

- Looks like there are several definitions for Kaiser window values, I get
  different results compared to Matlab.

- I use 32 bit float and integers because it's enough?.

  NOAA 15:
  NOAA 
  NOAA 18: 137.9125MHz

  Portadora AM: 2400Hz
  Amplitud: Escala de grises

  Cada palabra es 8 bits/pixel
  Dos lineas por segundo.
  4160 words/segundo.
  909 words utiles por linea
  Cada linea contiene las dos imagenes
  2080 pixeles/linea

  Cosas por línea:

  - Sync A: Onda cuadrada de 7 ciclos a 1040Hz
  - Space A:
  - Image A:
  - Telemetry A:
  - Sync B: Tren de pulsos de 842 de 7 ciclos????
  - Space B:
  - Image B:
  - Telemetry B:

  Procedimiento:

  - WAV a 11025Hz
  - Filtro pasa bajos
  - Resampleo a 9600Hz (4 veces más que la AM a 2400Hz)
  - Quedan 4 muestras por word de la AM de 2400Hz, cada muestra a 90 grados de
    diferencia de fase
  - Convierte a complejo y toma el módulo
  - Resampleo a 4160

## References

- [Digital Envelope Detection: The Good, the Bad, and the Ugly][1]: Lists some
  AM demodulation methods.

- [Hilbert Transform Design Example][2]: How to get the analytic signal.

- [Spectral Audio Signal Processing: Digital Audio Resampling][3].

- [Impulse Response of a Hilbert Transformer][4].

- [Spectral Audio Signal Processing: Kaiser Window][5].

- [How to Create a Configurable Filter Using a Kaiser Window][6],

[1]: https://www.dsprelated.com/showarticle/938.php
[2]: https://www.dsprelated.com/freebooks/sasp/Hilbert_Transform_Design_Example.html
[3]: https://ccrma.stanford.edu/~jos/resample/
[4]: https://flylib.com/books/en/2.729.1/impulse_response_of_a_hilbert_transformer.html
[5]: https://ccrma.stanford.edu/~jos/sasp/Kaiser_Window.html
[6]: https://tomroelandts.com/articles/how-to-create-a-configurable-filter-using-a-kaiser-window

[analytic signal]: https://en.wikipedia.org/wiki/Analytic_signal
