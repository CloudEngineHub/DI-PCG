# Customize DI-PCG

This document gives a brief introduction on how to use your own procedural content generation (PCG) model to train a diffusion model for efficient inverse problem.

### Overview
This document supports PCG models built with Blender using [Geometry Nodes](https://docs.blender.org/manual/en/latest/modeling/geometry_nodes/index.html). As a very popular procedural modeling tool, Blender Geometry Nodes provides a flexible and powerful way to build complex procedural models. Once you have your own PCG model built with Blender, you can use DI-PCG to train a diffusion model for efficient inverse problem, following the five steps below.

1. **Convert Blender PCG model into python codes.** [Infinigen](https://github.com/princeton-vl/infinigen) provides conveninent tools to transpile Blender PCG nodes into python codes.
2. **Modify python codes to include essential interfaces.** DI-PCG uses pre-defined functional interfaces to interact with procedural generator. You need to modify the transpiled python codes to be consistent.
3. **Generate training data.** DI-PCG provides a script to generate training data from PCG model. You can adjust the number of the generated data and the render configurations accordingly.
4. **Train diffusion model.** Adjust the training configurations accordingly, and launch the diffusion model training for your PCG. 
5. **Inference.** After training, you can use the trained diffusion model to perform efficient inverse given input images. Local gradio demo script is also provided for your convenience.

Next, we will introduce the above five steps in detail.

### Step 1: Convert Blender PCG model into python codes
Given Blender geometry node based procedural generator, you can use `./scripts/transpiler_script.py` from [Infinigen](https://github.com/princeton-vl/infinigen) to transpile the Blender PCG model into python codes. Specifically,

- Create a Blender file
- Click the "Scripting" tab in Blender
- Copy the code from `./scripts/transpiler_script.py` into the new script
- Select an object which has some geometry nodes on it
- Click the play button at the top of the script to run it
- The transpiled python codes will be saved to a file named `generated_surface_script.py`


### Step 2: Modify python codes to include essential interfaces
The transpiled python code is not ready to be used for DI-PCG yet. Some modifications are needed to be consistent with existing interfaces. See `./core/assets/dandelion.py` for an example. After getting the transpiled python codes which represent the geometry nodes, you need to:

- Implement a new class for the new procedural generator, inheriting the `AssetFactory` class
- Implement the `get_params_dict` function to define all the desired parameters, with their type and ranges.
- Implement the `sample_parameters` function to randomly sample a set of parameters from the defined ranges.
- Implement the `fix_unused_parameters` function to set unused parameters to their mid value.
- Implement the `update_parameters` function to update the parameters with the given values and dependency.
- Implement the `create_asset` function to create the procedural asset with the given parameters. Here, you can mainly use the content of `apply` function from the transpiled python codes.

### Step 3: Generate training data
To train a model, image-parameter paired data are required. DI-PCG provides a script to randomly sample the procedural generator and render multi-view images. To prepare the training data, run:
```
python ./scripts/prepare_data.py --generator your_own_pcg_name --save_root /path/to/save/training/data
```

### Step 4: Train diffusion model
DI-PCG provides a easy way to train a diffusion model for your PCG. First, adjust the training configurations in `configs/train/your_own_pcg_name.yaml` accordingly, and launch the diffusion model training simply by:
```
python ./scripts/train_diffusion.py --config ./configs/train/your_own_pcg_name.yaml
```

### Step 5: Inference
After training, you can find the trained diffusion model checkpoints in `./logs`. You can use the trained diffusion model to perform efficient inverse given input images. 

**To run the inference**, first update the `configs/demo/your_own_pcg_name.yaml` file with correct paths, and then run:
```
python ./scripts/sample_diffusion.py --config ./configs/demo/your_own_pcg_name.yaml
```
**To lanch the local gradio demo**, first modify the `app.py` to include your new PCG, then run:
```
python app.py
```

