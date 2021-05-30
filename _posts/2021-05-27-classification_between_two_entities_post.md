# Two Entity Discrimination with FastAI

![Data Example](images/entity-discrimination-data-example.png)

First, we need to download the images. In this example we will discriminate between two kinds of mushrooms – Fly Agaric (poisonous) and Champignon (edible). A Bing Image Search key is needed (free). In order to set the key run this in your terminal: `export AZURE_SEARCH_KEY=your_key_here`.
```
key = os.environ.get('AZURE_SEARCH_KEY', 'XXX')
mushroom_types = 'fly agaric','champignon'
path = Path('mushrooms')

if not path.exists():
    path.mkdir()
    for o in mushroom_types:
        dest = (path/o)
        dest.mkdir(exist_ok=True)
        results = search_images_bing(key, f'{o} mushroom')
        download_images(dest, urls=results.attrgot('contentUrl'))
```
While downloading from Bing, some of the images are broken and need to be removed:
```
fns = get_image_files(path)
failed = verify_images(fns)
failed.map(Path.unlink);
```

Now we need to prepare the dataset further. We are using the `ImageDataLoaders` to load the images and sort them into their categories based on folder structure. `valid_pct` means the percentage of images that would be included in the validation set (the rest would be used for training). `item_tfms` is the function that will resize each images (`Resize(224)` means that each image would be resized to 224x224 pixels). The `seed` is used to generate all the random operations during the training (image shuffling, crop position shuffling etc.). It allows us to demonstrate that the changes in training quality are due to architecture changes and not randomness.
```
from fastai.vision.all import *
path = Path('mushrooms')

dls = ImageDataLoaders.from_folder(path,
                                   get_image_files(path),
                                   valid_pct=0.2,
                                   seed=42,
                                   item_tfms=Resize(224)
                                  )
dls.show_batch()
```

Fine tune the `resnet50` model with our images.
```
learn = cnn_learner(dls, resnet50, metrics=error_rate)
learn.fine_tune(7)
```


As you can see, one of the worst predictions is a misslabeled image.

![Worst Predictions](images/entity-discrimination-worst-predictions.png)

![Confusion Matrix](images/logo.png)
