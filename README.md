# Godot Export
A workflow action to automatically export your Godot games. Supports standard and **Mono** builds!

## How it Works

### Automatic Exports
This action automatically reads your `export_presets.cfg` file to determine which builds to make. Whenever this action runs each of your defined exports will run. The resulting executables will be posted as zip files attached to the created release.

### Automatic Release
This action creates a release tagged with a [Semantic Version](https://semver.org/). The zip files containing the export results will be attached to this release. More info about releases can be found below.

## Configuration
### In Godot
Make sure that you have defined at least 1 export preset by going to `Project -> Export` in the Godot editor and creating a configuration. Also set the file name in the "Export Path" box. This action will not know how how to name your file without this. Notice how the below image has "win.exe" in the "Export Path" for my windows export. Your executables can be named however you like, as long as they include the appropriate extension `.exe`, `.x86_64`, etc.
![](docs/exports.png)

**NOTE**: The file extension for the Mac OSX export must be anything but `.zip` for versions of Godot before 3.2. If the Mac OSX export file extension is `.zip` for a Godot version earlier than 3.2, then your project source files will be exported instead of a Mac application. [This was a known issue with the Godot command line export](https://github.com/godotengine/godot/issues/23073). For reference, I used a `.z` file extension to make it work for my Mac OSX builds.

### Workflow
This action requires that your job utilizes Github's `actions/checkout` so that the source code is available for Godot to export the game. See the below [example workflow configuration](#example-workflow-configuration) for an example.

### Environment Variables
Since this action creates releases and uploads the zip file assets, you will need to supply the `GITHUB_TOKEN` environment variable. For an example on how to do this, see the below [example workflow configuration](#example-workflow-configuration). This environment variable is not needed if you set `create_release` to `false`.


### Inputs
- `godot_executable_download_url`
  - The **Linux Headless** version of Godot that you want to export your project with. For example, to use the current stable of version of Godot your value will be `https://downloads.tuxfamily.org/godotengine/3.1.2/Godot_v3.1.2-stable_linux_headless.64.zip`. If you do not use the Linux Headless version exporting will fail.
- `godot_export_templates_download_url`
  - The link to the `.tpz` archive of export templates. Can be found at `https://downloads.tuxfamily.org/godotengine`. The export templates must be for the same version of Godot that you are using in `godot_executable_download_url`. For example, the `godot_export_templates_download_url` that matches the `godot_executable_download_url` version is `https://downloads.tuxfamily.org/godotengine/3.1.2/Godot_v3.1.2-stable_export_templates.tpz`
- `relative_project_path`
  - The relative path to the directory containing your `project.godot` file. If your `project.godot` is at the root of your repository then this value should be `./`. Do _not_ include `project.godot` as part of this path.
- `base_version` default `0.0.1`
    - The version which new releases start at. The first release will be this version. After that, releases will automatically be 1 patch version ahead of the version of the latest release. To increment minor and major versions simply set the `base_version` to reflect your desired major and minor versions. When the `base_version` is set to a higher version than the last release version, the `base_version` will be used.
- `create_release` default `true`
  - Enable release creation. If `false`, exports will be available in folder `exports` in `relative_project_path`.
- `generate_release_notes` default `false`
  - Enable auto-generation of release notes based on commit history.

### Example Workflow Configuration
Below is a sample workflow configuration file utilizing this action. This example workflow would be defined in `.github/workflows/main.yml`. For more information about defining workflows [check out the workflow docs](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/configuring-a-workflow).

```yml
# Whenever a push is made to the master branch then run the job
on: 
  push:
    branches:
      - master

jobs:
  # job id, can be anything
  export_game:
    # Always use ubuntu-latest for this action
    runs-on: ubuntu-latest
    # Job name, can be anything
    name: Export Game Job
    steps:
      # Always include the checkout step so that 
      # your project is available for Godot to export
    - name: checkout
      uses: actions/checkout@v2.1.0
    - name: export game
      # Use latest version (see releases for all versions)
      uses: firebelley/godot-export@v1.4.0
      with:
        # Defining all the required inputs
        # I used the mono version of Godot in this example
        godot_executable_download_url: https://downloads.tuxfamily.org/godotengine/3.2.1/mono/Godot_v3.2.1-stable_mono_linux_headless_64.zip
        godot_export_templates_download_url: https://downloads.tuxfamily.org/godotengine/3.2.1/mono/Godot_v3.2.1-stable_mono_export_templates.tpz
        relative_project_path: ./
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Tips
I recommend creating a separate branch just for the purposes of running this action. Suppose I want the action to run on `master` pushes, but I am in the middle of working on game-breaking changes. Rather than push directly to master and create broken builds (and releases) you might want to consider some different approaches:
  - You could create a `release` branch that this action runs on. Then merge your `master` branch into `release` whenever you want to generate a release.
  - You could keep `master` as your release-generating branch, and do active development on a `dev` branch. Merge `dev` into `master` when you want to create a release.
