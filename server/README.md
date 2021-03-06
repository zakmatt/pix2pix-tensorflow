# pix2pix-tensorflow server

Host pix2pix-tensorflow models to be used with something like the [Image-to-Image Demo](https://affinelayer.com/pixsrv/).

This is a simple python server that serves models exported from `pix2pix.py --mode export`.  It can serve local models or use [Cloud ML](https://cloud.google.com/ml/) to run the model.

## Local

Using the [pix2pix-tensorflow Docker image](https://hub.docker.com/r/affinelayer/pix2pix-tensorflow/):

```sh
alias p2p-run="sudo docker run --rm --volume /:/host --workdir /host\$PWD --env PYTHONUNBUFFERED=x --env CUDA_CACHE_PATH=/host/tmp/cuda-cache --env HOME=/host\$HOME --publish 8000:8000 affinelayer/pix2pix-tensorflow"

# export a model to upload
p2p-run python export-example-model.py --output_dir models/example
# process an image with the model using local tensorflow
p2p-run python process-local.py \
    --model_dir models/example \
    --input_file static/facades-input.png \
    --output_file output.png
# run local server
p2p-run python serve.py --local_models_dir models
# test the local server
curl -X POST http://localhost:8000/example \
    --data-binary @static/facades-input.png >! output.png
```

If you open [http://localhost:8000/](http://localhost:8000/) in a browser, you should see an interactive demo, though this expects the server to be hosting the exported models available here:

- [edges2shoes](https://mega.nz/#!HtYwAZTY!5tBLYt_6HFj9u2Kxgp4-I36O4EV9r3bDP44ztX3qesI)
- [edges2handbags](https://mega.nz/#!Clg3EaLA!YW2jfRHvwpJn5Elww_wM-f3eRzKiGHLw-F4A3eQCceI)
- [facades](https://mega.nz/#!f1ZjmZoa!mCSxFRxt1WLBpNFsv5raoroEigxomDVpdi40aOG1KMc)

Extract those to the models directory and restart the server to have it host the models.

## Cloud ML

For this you'll want to generate a service account JSON file from https://console.cloud.google.com/iam-admin/serviceaccounts/project (select "Furnish a new private key").  If you are already logged in with the gcloud SDK, the script will auto-detect credentials from that if you leave off the `--credentials` option.

```sh
# upload model to google cloud ml
p2p-run python upload-model.py \
    --bucket your-models-bucket-name-here \
    --model_name example \
    --model_dir models/example \
    --credentials service-account.json
# process an image with the model using google cloud ml
p2p-run python process-cloud.py \
    --model example \
    --input_file static/facades-input.png \
    --output_file output.png \
    --credentials service-account.json
```

