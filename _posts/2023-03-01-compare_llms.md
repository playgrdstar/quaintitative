---
layout: default
title:  Comparing Prompts for Different Large Language Models (Other than ChatGPT)
description: Comparing Prompts for Different Large Language Models (Other than ChatGPT)
date:   2023-03-01 00:00:01 +0000
permalink: /compare-llms/
category: Research
---
## Comparing Prompts for Different Large Language Models (Other than ChatGPT)
### And notes on using Hugging Face Spaces, Inference APIs and Gradio Blocks

Prompts are what we type into the text box or provide as inputs when we want ChatGPT or other large language models (LLM) to generate a response. ChatGPT (or say the text-davinci model via OpenAI's APIs) works quite nicely with simple prompts. However, better responses can be obtained if we craft our prompts carefully (typically referred to as "prompt engineering").

The [Prompt Engineering Guide][1] is an interesting repository (not by me) that contains useful information on prompt engineering, in particular their guides on prompt engineering. However, the prompts and corresponding responses shown in the [Prompt Engineering Guide][1] are for ChatGPT. 

I was curious to see what the responses to these prompts would look like for other LLMs. So, I created a [Hugging Face Spaces][2] space that allows you to compare the different prompts and responses for different LLMs. Feel free to play around with the space. I have included common and popular LLMs such as Flan-T5, GPT and Bloom variants. The key prompts that the [Prompt Engineering Guide][1] illustrated are provided as examples in the space.

It's quite simple to get such a space up and going thanks to Hugging Face. The code for the space is available [here][3], but I thought it would be useful to run through some of the key steps involved as my own notes (for future reference) and for folks who are not familiar with Hugging Face Spaces.

TLDR: Get a Hugging Face account. Create a Space. Clone the repo of the Space added. Add an app.py file using the Inference API and Gradio. Push it to the repo. 

### Creating a Space

We shall start by assuming you have a Hugging Face account. Once you have an account you can get an API token from the [Access Tokens page][5]. This token will be used to authenticate your requests to the Hugging Face API. Create one now and save it somewhere.

Now go to the Spaces page [here][4]. Click on "Create new Space" (top right).

Provide the relevant details. We can choose between options such as using Gradio or Streamlit. For this example, I chose Gradio. The other fields are self-explanatory.

Once the Space is created, you will see a series of instructions on how to get started. The first step is to clone the repo of the Space created to your local machine (assuming you have git installed).

After you have done so, you will need to add an app.py file to the repo. The app.py file is the file that contains the code that will be executed when you run the Space.

To recreate this example, there are two approaches that you can take. You can load the neccessary pre-trained LLMs in your Space, and then use these loaded LLMs to generate responses. The code in my Space includes lines (commented out) that adopt this approach if you are interested.

### Using the Hugging Face Inference API

However, what I did was to use the Hugging Face Inference API to generate responses. This approach is much simpler and does not require you to load the LLMs in your Space. Doing this is pretty straightforward. I have written a lot of other code to do some checks and also present the results, but in general, all you really need to use the Inference API is the following code:

(Do note that the HF_READ_API_KEY corresponds to the API token that you created earlier, which you will need to save as a secret in the Settings page of your Space. For the parameters that you provide in the header, you can read more about them [here][6].)

```
# use the requests module to send a request to the Hugging Face inference API
import requests 

# compose the header, temperature will detemine how random the response will be, max_tokens is the max number of tokens in the response, and max_time is the max time in seconds that the API will wait for a response

headers = {f"Authorization": f"Bearer {HF_READ_API_KEY}", 
            "wait_for_model": "true", 
            "temperature": str(temperature), 
            "max_tokens": str(max_tokens),
            "max_time": str(120)}

# compose the API url for the model you want to use
model_name = "google/flan-t5-small"
model_api_url = f"https://api-inference.huggingface.co/models/{model_name}"

# compose the payload with the query which is basically the prompt
query = "The sky is "
payload = {"inputs": query}

# send the request to the API
response = requests.post(model_api_url, headers=headers, json=payload)

# check if the response is successful (with a code of 200), if not, try again.
while response.status_code != 200:
    response = requests.post(model_api_url, headers=headers, json=payload)
```

### Introduction to Gradio

The code in the app.py file for the user interface uses the Gradio library. Gradio is a Python module that allows you to create user interfaces for your models without much effort. The easiest way is to use the Interface function. However, for greater control, I used Blocks. The code for the user interface is pretty straightforward. I have included comments in the key code extracts below to explain what is going on.

```
# start a Block
with gr.Blocks(css='style.css') as demo:
    # this allows you to add some text to the interface
    gr.HTML("""
    ...
    """)

    # start a new column
    with gr.Column(elem_id="col-container"):  
        # start a new row and add a dropdown menu to select the model
        with gr.Row(variant="compact"):
            model_name = gr.Dropdown(
                 model_list, 
                 label="Select model",
                 value=model_list[0],
                 ).style(
                container=False,
            )
 
            # add sliders to select the temperature and max tokens
            temperature = gr.Slider(
                0.1, 100.0, value=1.0, label="Temperature",
            ).style(
                container=False,
            )
            max_tokens = gr.Slider(
                10, 250, step=1, value=100, label="Max. tokens (in output)",
            ).style(
                container=False,
            )
            # add a checkbox to check if the output is truncated
            check_truncated = gr.Checkbox(
                label="Check for truncated output",
                value=False,
            ).style(
                container=False,
            )

        # start a new row and add a textbox to enter the prompt
        with gr.Row(variant="compact"):
            prompt = gr.Textbox(
                label="Enter your prompt",
                show_label=False,
                # max_lines=2,
                placeholder="Select your prompt from the examples below",
            ).style(
                container=False,
            )
            process = gr.Button("Generate").style(full_width=False)
        
        # start a new row and add a textbox to display the output
        with gr.Row():
            output=gr.Textbox(
                label="LLM output",
                show_label=True)
            
        # start a new row and add examples. Note the inputs parameter, which will determine where the example prompts will be added to when you select them
        gr.HTML("""
        <div>
        <h4 style="font-weight: 50; font-size: 14px; margin-bottom:0px; margin-top:0px;">
        Prompt examples. Select the prompt you would like to test, and it will appear (properly formatted) in the input box above.
        </h4>
        </div>
        """)
        with gr.Tab("Introduction"):
            example_set_1 = gr.Examples(label = 'Simple Prompt vs. Instruct then Prompt.', 
                                        examples=["The sky is ", "Complete the following sentence: The sky is ",],
                                        inputs=[prompt])
            example_set_2 = gr.Examples(label = 'Few Shot Prompt.', 
                                        examples=["This is awesome! // Positive\nThis is bad! // Negative\nWow that movie was rad! // Positive\nWhat a horrible show! //",],
                                        inputs=[prompt])
            ...
```

And that's it. Pretty straightforward, right? 


[1]:    https://github.com/dair-ai/Prompt-Engineering-Guide
[2]:	https://huggingface.co/spaces/playgrdstar/compare-llms
[3]:    https://huggingface.co/spaces/playgrdstar/compare-llms/tree/main
[4]:    https://huggingface.co/spaces
[5]:    https://huggingface.co/settings/token
[6]:    https://huggingface.co/docs/api-inference/detailed_parameters 
