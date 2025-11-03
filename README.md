# Nori to Mitsuba Converter

A simple python script that converts Nori scenes to Mitsuba scenes (XML to XML). The converter is very basic and only supports a limited set of features, but can be very useful for quick comparisons.

### Usage

`python3 nori_to_mitsuba.py <nori_file> [<mitsuba_file>]`

### Example

Running

`python3 nori_to_mitsuba.py scene_nori.xml scene_mitsuba.xml`

will generate a Mitsuba scene file in the same directory: `scene_mitsuba.xml`. 

If no output path is specified, then the converter will use the input path and rename it:

- If the input file contains the string `nori`, then `nori` will be replaced with `mitsuba`. Example: `very_cool_nori_scene.xml` -> `very_cool_mitsuba_scene.xml`.
- Otherwise, the output file will be named `<input_file>_mitsuba.xml`. Example: `scene.xml` -> `scene_mitsuba.xml`.

The original file will not be modified.

### Features

This script is very basic and only supports the following features:

- Translation of basic scene information:
    - Integrator, sampler, resolution, etc.
    - Path integrators (mats, mis) get mapped to `path`, and direct integrators (ems, mats, mis) get mapped to `direct`
    - The camera gets mirrored to match the different coordinate system
- Translation of objects via a simple remapping of tags and attributes:
    - `mesh` -> `shape`
    - `color` -> `rgb`
    - `albedo` -> `reflectance` (diffuse bsdf)
    - ...
- Everything else is left unchanged

New tag and attribute mappings can easily be added to the respective `tag_map` and `attrib_map` dictionaries:

```python
tag_map = {
    "mesh": "shape",
    "color": "rgb",
    # "nori_tag": "mitsuba_tag"
}
```

> [!IMPORTANT]
> The attribute mappings are not context-dependent, i.e. they get applied to all tags, anywhere in the file. For example, if you implement a different bsdf with a `albedo` attribute, then this will also be remapped to `reflectance`, which might not be recognized by Mitsuba.

Full list of attribute mappings:

| Category     | Attribute Key | Nori Value     | Mitsuba Value   |
| ------------ | ------------- | -------------- | --------------- |
| BSDF         | type          | disney         | principled      |
| BSDF         | type          | mirror         | conductor       |
| BSDF         | type          | transparent    | null            |
| BSDF         | type          | blended        | blendbsdf       |
| Texture      | type          | image_float    | bitmap          |
| Texture      | type          | image_color    | bitmap          |
| Diffuse BSDF | name          | albedo         | reflectance     |
| Texture      | name          | interpolation  | filter_type     |
| Texture      | name          | extension      | wrap_mode       |
| Pointlight   | name          | power          | intensity       |
| Disney BSDF  | name          | specularTint   | spec_tint       |
| Disney BSDF  | name          | sheenTint      | sheen_tint      |
| Disney BSDF  | name          | clearcoatGloss | clearcoat_gloss |
| Disney BSDF  | name          | subsurface     | flatness        |

You might need to adjust the `attrib_map` dictionary to fit your needs.

### Limitations

- Mitsuba doesn't have a simple microfacet BSDF. I think it can be achieved with a blend of a diffuse BSDF and a rough dielectric BSDF, but due to the simple remapping of tags and attributes, this is not supported.
- Remapping is not context-dependent. For example, any tag that contains `name="albedo"` as an attribute will be remapped to `name="reflectance"`. This might not be the desired behavior in all cases.
- General scene info (integrator, sampler, camera, etc.) is handled differently than object info. This means that it is not as flexible as it could be. For example, if you want to have a parameter in the integrator (e.g. `minDepth`), then you would have to change the code to accout for this.
- Mitsuba doesn't recognize the `rotate` tag in a `transform`.
- Pointlights are supported but Mitsuba uses a different unit for intensity (power per unit steradian instead of total power). Due to the simple remapping, there is no conversion of the intensity.