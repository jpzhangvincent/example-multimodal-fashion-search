# Multimodal Fashion Search with Jina

![](./.github/images/demo.gif)

Multimodal search lets you use one type of data (in this case, text) to search another type of data (in this case, images). This example leverages core Jina technologies that make it simpler to build and run your search, including:

- **[DocumentArray](https://docarray.jina.ai)** - let's us concurrently process Documents and push/pull them between machines. Useful for creating embeddings on remote machine with GPU and then indexing and querying locally
- **[Jina Hub](https://hub.jina.ai)** Executors, so we don't have to manually integrate deep learning models
- **[Jina Client](https://docs.jina.ai/api/jina.clients/)**, so we don't have to worry about how best to format the REST request
- **[PQLite](https://hub.jina.ai/executor/pn1qofsj)** allowing us to pre-filter results by season, price, rating, etc

The front-end is built in [Streamlit](https://streamlit.io/).

## What do you want to do?

Your instructions will be different based on whether you want to:

- Just run the demo on your own machine - this pulls a pre-indexed DocumentArray from the cloud so you don't need to index anything. You just need to build an index and search the pre-existing data.
- Adapt the demo for your own dataset - in this case, you'll want to index your own data and then search that

## Run fashion demo

1. `pip install -r requirements.txt`
2. `cd run_fashion_demo`
3. `python create_index.py` to pull the pre-embedded DocumentArray from the cloud and create an index (this saves you having to do compute-heavy embedding locally)
4. `cd ../backend-<format>` (where `<backend>` is `image` or `text`)
5. `python app.py -t <task>` to start the search server(s) (where `<task>` is either `search` for a RESTful API or `search_grpc` to do a quick test search in the terminal)
6. `cd ../frontend`
7. `streamlit run frontend.py` to start things in your web browser

> If you any filenames starting with `x_` then they're just something we use internally to build the initial DocumentArray or other tasks the end user doesn't need to worry about. But they're there for your reference if you want them!

## Adapt the demo for your own needs

### Setup

`pip install -r requirements.txt`

### Download and clean up data

You'll want to create your own `get_data.py` or some other way to process your dataset. We tend to keep all dataset processing code in a file like this since processing logic varies from dataset to dataset. In fashion search's [`get_data.py`](./run_fashion_demo/x_get_data.py) we're cleaning up a CSV file to prevent malformed rows causing trouble later on.

### Create embeddings and index your data

This will create embeddings for all images using CLIPImageEncoder, and then store them on disk (with metadata) with PQLiteIndexer.

1. `cd indexer`
2. `python app.py <number_of_docs_to_index>`

By default the number of docs to index is set to 99,999,999

### Copy over columns

After indexing you'll have a file called `columns.json` in your `indexer` directory. Copy this to the `backend-` directories you want to work with. This will let the user filter by things like price, color, year, etc (based on what options you present in your front-end). This will overwrite the existing `columns.json` file(s) which are the ones from the fashion search.

### Run search backends

We have two backends:

- [Text-to-image](./backend-text): Input text, get images returned
- [Image-to-image](./backend-image): Input image, get images returned

To run either or both of those:

1. `cd ../backend-<format>` (where `<backend>` is `image` or `text`)
2. `python app.py -t <task>` to start the search server(s) (where `<task>` is either `search` for a RESTful API or `search_grpc` to do a quick test search in the terminal).

> For RESTful interfaces, both backends use different ports so you can run both at once.

### Run frontend

1. Open a new terminal window/tab, return to same directory
2. `cd frontend`
3. `streamlit run frontend.py`

## With Docker-compose

1. First index the data as stated above
2. In the repo's root directory, run `docker-compose up` 

## Tips

- Index using the [small dataset](https://www.kaggle.com/paramaggarwal/fashion-product-images-small), then swap out the data directory for that of the [hi-res dataset](https://www.kaggle.com/paramaggarwal/fashion-product-images-dataset) for nicer-looking results.

## Debugging

### Search using gRPC

This is a good way to test quickly, instead of going through frontend. It'll take either a string ("shoes") or one of the images from the dataset (depending on whether you do image or text search) and use that as a query.

1. Index your data
2. `cd backend-text` or `cd backend-image`
3. `pip install -r requirements.txt`
4. `python app.py -t search_grpc`

It should then print out the `uri`s of the matching Documents.
