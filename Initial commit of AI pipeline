# 1. Essential Installs
!pip install -q diffusers transformers accelerate gradio segment_anything opencv-python

import torch
import gradio as gr
from diffusers import StableDiffusionInpaintPipeline
from PIL import Image

# 2. Load the Model with optimized settings
pipe = StableDiffusionInpaintPipeline.from_pretrained(
    "runwayml/stable-diffusion-inpainting",
    torch_dtype=torch.float16,
).to("cuda")

def final_try_on(input_data, prompt):
    # Prepare Image
    init_image = input_data["background"].convert("RGB").resize((512, 512))
    
    # Extract the Red Brush Mask
    mask_layer = input_data["layers"][0]
    mask_image = mask_layer.getchannel('A').point(lambda x: 255 if x > 0 else 0).convert("RGB").resize((512, 512))
    
    # Use strength=1.0 and a higher guidance_scale (15.0) 
    # to force the AI to overwrite the original suit completely.
    output = pipe(
        prompt=f"{prompt}, high quality, cinematic lighting, realistic fabric texture, 8k",
        negative_prompt="tie, suit, blazer, old clothes, blurry, bad anatomy, face distortion",
        image=init_image, 
        mask_image=mask_image,
        strength=1.0,           # 1.0 = Ignore the old pixels completely
        guidance_scale=15.0,    # 15.0 = Follow the prompt very strictly
        num_inference_steps=40  # More steps for higher realism
    ).images[0]
    
    return output

# 3. The UI
with gr.Blocks(theme=gr.themes.Soft()) as demo:
    gr.Markdown("# Professional AI Virtual Try-On")
    gr.Markdown("### **Crucial Step -** Brush over the **entire** suit area, including the shoulders and chest.")
    
    with gr.Row():
        img_input = gr.ImageEditor(
            label="Upload & Brush Over Suit", 
            type="pil", 
            brush=gr.Brush(colors=["#FF0000"], default_size=40)
        )
        img_output = gr.Image(label="New Outfit Result")
        
    prompt_text = gr.Textbox(label="Describe your new outfit", value="A casual bright red t-shirt")
    btn = gr.Button("Generate", variant="primary")
    
    btn.click(fn=final_try_on, inputs=[img_input, prompt_text], outputs=img_output)

demo.launch(share=True, debug=True)
