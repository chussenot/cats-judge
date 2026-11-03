# Image sources

Every photo under `images/` is a JPEG re-encoded from its source with the
longer side at most 512 pixels and the shorter side at least 280, the
limits the System One wire takes, EXIF orientation applied and stripped.

| File | Size | Source |
|---|---|---|
| `tench.jpg` | 426×320 | Imagenette (fast.ai, Apache-2.0), val/n01440764/ILSVRC2012_val_00009111.JPEG |
| `english-springer.jpg` | 426×320 | Imagenette (fast.ai, Apache-2.0), val/n02102040/ILSVRC2012_val_00004650.JPEG |
| `cassette-player.jpg` | 426×320 | Imagenette (fast.ai, Apache-2.0), val/n02979186/ILSVRC2012_val_00008651.JPEG |
| `church.jpg` | 325×320 | Imagenette (fast.ai, Apache-2.0), val/n03028079/ILSVRC2012_val_00003351.JPEG |
| `french-horn.jpg` | 396×320 | Imagenette (fast.ai, Apache-2.0), val/n03394916/ILSVRC2012_val_00001492.JPEG |
| `garbage-truck.jpg` | 426×320 | Imagenette (fast.ai, Apache-2.0), val/n03417042/ILSVRC2012_val_00002210.JPEG |
| `gas-pump.jpg` | 320×426 | Imagenette (fast.ai, Apache-2.0), val/n03425413/ILSVRC2012_val_00000732.JPEG |
| `parachute.jpg` | 426×320 | Imagenette (fast.ai, Apache-2.0), val/n03888257/ILSVRC2012_val_00001440.JPEG |
| `persian-98.jpg` | 375×500 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id Persian_98 |
| `sphynx-59.jpg` | 500×396 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id Sphynx_59 |
| `abyssinian-57.jpg` | 375×500 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id Abyssinian_57 |
| `british-shorthair-269.jpg` | 482×512 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id British_Shorthair_269 |
| `sphynx-239.jpg` | 451×512 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id Sphynx_239 |
| `british-shorthair-89.jpg` | 416×500 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id British_Shorthair_89 |
| `birman-62.jpg` | 500×333 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id Birman_62 |
| `ragdoll-45.jpg` | 500×367 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id Ragdoll_45 |
| `british-shorthair-70.jpg` | 500×334 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id British_Shorthair_70 |
| `persian-265.jpg` | 364×369 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id Persian_265 |
| `siamese-73.jpg` | 333×500 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id Siamese_73 |
| `bombay-60.jpg` | 500×281 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id Bombay_60 |
| `newfoundland-31.jpg` | 500×334 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id newfoundland_31 |
| `pug-57.jpg` | 500×333 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id pug_57 |
| `american-bulldog-80.jpg` | 500×333 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id american_bulldog_80 |
| `great-pyrenees-22.jpg` | 500×342 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id great_pyrenees_22 |
| `havanese-97.jpg` | 500×333 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id havanese_97 |
| `staffordshire-bull-terrier-30.jpg` | 357×500 | Oxford-IIIT Pet (CC BY-SA 4.0), test split image_id staffordshire_bull_terrier_30 |

- **Oxford-IIIT Pet** (Parkhi, Vedaldi, Zisserman, Jawahar, 2012), test
  split as published on the Hub at
  [timm/oxford-iiit-pet](https://huggingface.co/datasets/timm/oxford-iiit-pet),
  [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/). The
  `image_id` is the dataset's.
- **Imagenette** (fast.ai), the 320-pixel validation split from
  [github.com/fastai/imagenette](https://github.com/fastai/imagenette),
  Apache 2.0 as the repository states; the images are a subset of
  ImageNet's validation set, named by their ImageNet file.

The meme at the top of the README, `docs/is-this-a-cat.jpg`, is
`british-shorthair-89.jpg` with a caption and the verdict the model gave it,
so it carries the same CC BY-SA 4.0 licence as its source.
