# Sickhowl Weather Editor for Anomaly

A simple weather editor written in Civet to easily parse and manipulate S.T.A.L.K.E.R. Anomaly weather files.

## Requirements

- [Node.js (LTS)](https://nodejs.org/en)
  - [nvm](https://github.com/nvm-sh/nvm) is highly recommended to manage your local Node.js version.
- [`magick`](https://imagemagick.org/script/download.php#:~:text=an%20iOS%20application.-,Windows%20Binary%20Release,-ImageMagick%20runs%20on)
- [Civet VSCode Extension](https://civet.dev/integrations#vscode)

If you're on Linux, this repository already includes the `magick` binary file.

## Usage

Open the `index.civet` file and get familiarized with it.

There are a couple of examples there on how to iterate over all available weathers in your mod or just one.

You can also iterate over different time ranges (i.e. from 09:00 to 09:59 including all times in between).

Once you are comfortable enough with your changes, run the `npm run start` command and it'll apply the changes to the weather LTX files.

> [!CAUTION]
> Make sure you make a backup of your weather files before running this command!
> This tool won't do that for you (maybe in the future).

---

All available types are included under the `Types` namespace, which can be imported from the editor file.

A `utils` import is also available with utility functions (mostly for math stuff like `clamp` etc).

Finally, right now the only "fancy" feature from the weather editor is being able to grab the average sky texture color to be used as a parameter for `fog_color` and so on. Keep in mind this feature will keep the `png` files from the library used to grab colors until you call the `cleanUpSkyTextures()` method

## Examples

```ts
{ WeatherEditor, WeatherTimeHelper } from ./editor.civet
{ clamp } from ./utils.civet

root := "C:/Anomaly/MO2/mods/Fix Atmospherics Stupid Clear Weather"
timeRange := ['09:00', '14:59'] as const

weatherEditor := new WeatherEditor { root }

// Grab all weather names available (i.e. `w_clear1`, `w_clear2`...)
allWeathers := weatherEditor.fetchAllWeathersNames()

// Iterate over all of them, this means we'll be changing all the weather files
for weatherName of allWeathers
  // Iterate over all weather times (i.e. `00:00`, `00:30`...)
  result := await weatherEditor.iterateOver weatherName, (def, time) => {
    // Skip weathers outside the time range
    unless WeatherTimeHelper.inRange time, timeRange then return

    // Increase the `far_plane` and `fog_distance` in 200
    def.far_plane = clamp(def.far_plane + 200, 200, 1000)
    def.fog_distance = clamp(def.fog_distance + 200, 200, 1000)

    // Read the average sky texture colors
    skyTexture := def.sky_texture
    skyTextureColors := await weatherEditor.getSkyTextureAvgColor skyTexture

    // If an average color was found, map it to the expected engine format and used them as the `fog_color`
    if skyTextureColors
      defSkyColor := def.sky_color
      def.fog_color = skyTextureColors |> .map (value) => Number(Math.max(0.01, value - 0.1).toFixed(2))
      console.info 'Setting fog color to', (def.fog_color.join ', '), 'for', skyTexture
  }
  
  // Write the result of the changes above to the weather file
  weatherEditor.writeIterateResultToWeatherFile result

// Clean up the sky textures `png` files when we're all done
weatherEditor.cleanUpSkyTextures()
```
