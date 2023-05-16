# Embedding-inspector extension version version 2.83 - 2023.01.13

for ![AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Extensions)

With this extension you can inspect internal/loaded embeddings and find out which embeddings are similar, and you can mix them to create new embeddings.

Inspired by [Embeddings editor](https://github.com/CodeExplode/stable-diffusion-webui-embedding-editor.git) and ![Tokenizer](https://github.com/AUTOMATIC1111/stable-diffusion-webui-tokenizer.git) extensions.

# What's new

v2.0: SD2.0 and multi-vector support 

v2.1: Entering embedding ID instead of name is now supported, for example you can enter "#2368" instead of "cat"

v2.2: Entering a step value (like 1000) is now supported. This is needed only if you will continue training this embedding. Also, step and checkpoint info for loaded embeddings are now displayed.

v2.3: Added "List loaded embeddings" button

v2.4: Added "Concat mode" option. In this mode, embeddings will be just combined instead of being mixed. For example, "mona" and "lisa" can be combined into a single embedding "monalisa" which will contain 2 vectors, and the result will be the same as having "mona lisa" in the prompt, but with a single keyword.

v2.5 Added a mini tokenizer. You can select "Send IDs to mixer" option to automate converting a short prompt to an embedding.

v2.52 Added an experimental eval feature. Text entered in Eval box will be evaluated and applied to the saved embedding. Not exactly useful, but see bottom of this page for usage.

v2.53 Added graph for saved embedding. (temporarily disabled in v2.531)

v2.532 Added  magnitude, min, max to displayed embedding info. Not much use but most internal embeddings seem to have around 0.3-0.4 magnitude. Added "combine as 1-vector" option. See bottom of this page for details.

v2.533 Added "reset mixer" button

v2.54 Bugfix for upper-case embedding names. Also disabled showing checksum when listing loaded embeddings

v2.55 Remove zeroed vectors (as an option in the script REMOVE_ZEROED_VECTORS = True)

v2.56 Showing graph of saved embedding is now enabled

v2.57 Added graph for inspected embedding, and button for saving the vector to text file (saved in webui root folder)

Added 'Eval presets' dropdown list, which lets you choose one of the 7 example eval strings, and 'Save for ALL presets' option (careful as this will save 8 embeddings, see screenshot at the bottom of this page).

v2.8 Bugfix for saved embeddings not reloading issue. 

Some terminology fixes in UI, and SHOW_SIMILARITY_SCORE as an option in script, default is False, change to = True to enable it.

Increased number of mixer lines, click on arrow to show/hide more lines. 

Added 'Binary' eval preset, and made vec_mag, vec_min, vec_max variables available.

# Manual Installation

Download ![embedding-inspector-main.zip](https://github.com/tkalayci71/embedding-inspector/archive/refs/heads/main.zip) and extract into extensions folder.

# Usage

1) Enter a token name into "Text Input" box and click "Inspect" button. Only the first token found in the text input will be processed. Below, some information about the token will be displayed, and similar embeddings will be listed in the order of their similarity. This is useful to check if a word is in the token database, find internal tokens that are similar to loaded embeddings, and also to discover related unicode emojis.

![image](screenshots/screenshot1.jpg)
![image](screenshots/screenshot4.jpg)

2) Enter one or more token names in the "Name 0", "Name 1"... boxes, adjust their weights with "Multiplier" sliders, enter a unique name in "Filename" box, click "Save mixed" button. This will create a new embedding (mixed from the given embeddings and weights) and save it in the embeddings folder. If the file already exists, "Enable overwrite" box must be checked to allow overwriting it. Then, you use the filename as a keyword in your prompt.

![image](screenshots/screenshot2.jpg)
![image](screenshots/screenshot3.jpg)

3) Enter a short prompt in mini tokenizer text box, select "Send IDs to mixer" option, click "Tokenize". In the mixer section IDs will have been copied and "Concat mode" checked. Adjust multiplier and global multiplier sliders if necessary, enter a filename and click "Save mixed" button. Then use the filename as a keyword in your prompt.

![image](screenshots/screenshot5.jpg)
![image](screenshots/screenshot6.jpg)
![image](screenshots/screenshot7.jpg)

# Background information

Stable Diffusion contains a database of ~49K words/tokens, and their numerical representations called embeddings. Your prompt is first tokenized using this database. For example, since the word "cat" is in the database it will be tokenized as a single item, but the word "catnip" is not in the database,  so will be tokenized as two items, "cat" and "nip". 

New tokens/concepts can also be loaded from embeddings folder. They are usually created via textual inversion, or you can download some from [Stable Diffusion concepts library](https://huggingface.co/sd-concepts-library). With Embedding-inspector you can inspect and mix embeddings both from the internal database and the loaded database.

# Eval feature

Embeddings consist of 768 or 1024 numbers, these numbers determine the generated image, but what each number controls is a mystery. With eval feature you can zero/modify some of these numbers to see what happens.

Enter an embedding name like "cat" in "Name 0" box, type a filename like "evaltest" and check "enable overwrite", enter the eval string in "Eval" box, click "save mixed".  You can check log for errors, and also inspect "evaltest" to see that the values have changed. Then generate the image in txt2img tab with the prompt "evaltest" to see the effect.

In the Eval string, use v as the original vector. Torch and math functions are available. Also following variables are available: vec_mag: magnitude of the vector, vec_min: minimum value in the vector, vec_max: maximum value in the vector

Examples:

Eval "v*2" multiplies by 2, increasing the strength of the embedding

Eval "v-vec_min" shifts all numbers up to the positive range, seems to have no effect.

Eval "torch.relu(v)" zeroes all negative values.

Eval "torch.abs(v)" makes all values positive.

Eval " v/vec_mag" normalizes the vector (error if magnitude is zero)

Eval " = torch.ceil(v)" rounds all values

If the Eval string starts with "=", evaluation will be done item-wise. Here available variables are : v=original value, i=item no (0:768 or 0:1024), maxi=item count (768 or 1024), n=vector no, maxn=vector count. Also, original values can be accessed as tot_vec[n,i] 


Eval " = v * (i<100)" zeroes items after 100th

Eval " = v * (i<maxi//2)" zeroes items in the upper half.

Eval " = v * (i>100 and i<200)" zeroes all items except between 100th and 200th.

Eval " = v * (i<400 or i>500)" zeroes all items between 400th and 500th.

Eval " = v * (i<300) * (n==0) + v * (i>300) * (n==1)" zeroes different parts of vectors 0 and 1 (in concat mode, see screenshot below)

![image](screenshots/screenshot8.jpg)
![image](screenshots/00000-2687304813-evaltest.jpg)

# Combine as 1-vector option

This option sums the final vectors into one vector, makes sense when used with eval feature. For example, to extract one of the vectors of a multi-vector embedding, you can use "=v*(n==2)" which zeroes all vectors but vector#2

Another case is to combine different parts of two embeddings as one, for which you can use an eval string like "=v* (n==0) * (i<300)+v * (n==1) * (i>=300)"

![image](screenshots/screenshot9.jpg)
![image](screenshots/00007-563623717-catdog.jpeg)


# Save for ALL eval presets

![image](screenshots/eval_presets.jpg)
