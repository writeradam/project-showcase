# llm-researcher
This web (&amp; file) research agent delivers seamless data collection in response to POST requests. It has a synchronous mode and an asynchronous mode.

## How it Works
The `llm-researcher` is a LangGraph implementation of a ReAct agent paradigm. It works in two modes:
* Synchronous: the app receives POST request, runs agent and returns results
* Asynchronous: the app receives POST request with additional parameters `id` and `webhook_url`, triggers the agent as a background process and responds immediately with `status` "processing"; when the agent is finished, it sends the response and `id` to the provided webhook URL

Available tools:
- SERP API by BrightData
- Web Scraper by BrightData (Web Unlocker) or Apify (RAG Web Browser)

## Prerequisites
Prerequisites depend on the your chosen model & tools. Typical prerequisites include:
* `OPENAI_API_KEY`
* `FIREWORKS_API_KEY`
* `BRIGHTDATA_API_KEY`
* `APIFY_API_KEY`

See the **Inputs for the `/run` endpoint** section for more information.

**Optional Prerequisites**
* `APIFY_SCRAPER_URL`: Use the following URL to open the RAG Web Browser actor in standby mode: `https://rag-web-browser.apify.actor/search`. To set up your actor, we recommend that you create a Task through Apify's UI, which efficiently specifies RAM and other shared parameters.  
* `APIFY_DATASET_LIMIT` (defauls to 100): This parameter sets the max number of past-run datasets to check before attempting to scrape the given URL

## How to Run
Multiple workers:
```commandline
gunicorn main:app -b=0.0.0.0:8001 --workers=2 --timeout 60 -k uvicorn.workers.UvicornWorker
```

Docker:
```commandline
docker build -t llm-researcher:latest .
docker save -o llm-researcher.tar llm-researcher:latest
docker run -it -p 8000:8000 -e WORKERS=2 --rm --name llm-researcher llm-researcher:latest 
```

## Question for Lukas ##
#--restart on-failure puvodne patrilo k Docker kodu, ale myslim si, ze to bylo mylne

## Troubleshooting ##

Trouble running the code? These fixes may help.

### Running unstable HTTPS3 for RUST ###

When you run the code, you may get this message:
```
error: The http3 feature is unstable, and requires the RUSTFLAGS='--cfg reqwest_unstable' environment variable to be set.
```
This means that `http3` support in the Rust dependency called `reqwest` is not yet stable. By default, Rust won't compile unstable features unless you explicity allow them.

You can allow unstable features in a few ways. 

#### How to allow unstable Rust features in Docker ####
If you're running the code in Docker, add an `ENV` line before your build step in the `Dockerfile`.

```dockerfile
FROM rust:1.80

# Set RUSTFLAGS globally
ENV RUSTFLAGS="--cfg reqwest_unstable"

WORKDIR /app
COPY . .

RUN cargo build --release
CMD ["./target/release/myapp"]
```
This way, all cargo build commands inside your container will use the unstable flag.

If you're trying to run code locally without changing the repo, then you can prefix commands:

```bash
docker build --build-arg RUSTFLAGS="--cfg reqwest_unstable" -t myapp .
```
### pip commands on macOS and Linux ###
If you run the Docker code on macOS or Linux, you may need to add `pip` to your shell config: 
```bash
alias pip3="pip"
```
Then reload:
```bash
source ~/.bashrc
```

## How to Deploy

### Native Python service on ZerOps - PREFERRED OPTION
Setup ZerOps `zcli` and run the following. It will take configuration from the `zerops.yml` file.
```commandline
zcli push
```
Choose the `n8n` project and `agentapp` service.

You can start and initialize the service via enviornment variables through the ZerOps UI.

### Docker on ZerOps
On ZerOps:
```commandline
zcli push
```
Choose the `n8n` project and `llmresearcher` service.

To set up the `llmresearcher` service, input:
```commandline
services:
  - hostname: llmresearcher
    type: docker@latest
    verticalAutoscaling:
      cpuMode: SHARED
      cpu: 3
      ram: 1
```
## Inputs for the `/run` endpoint ##
* `query` (string; required): query/task
* `model` (string; required): specify the model provider and model name separated by `:`
  * supported providers: `openai`, `fireworks`
  * examples: `openai:gpt-4o-mini`, `fireworks:accounts/fireworks/models/deepseek-v3` etc.
* `agent_implementation` (string; optional; defaults to "react:langgraph"). Currently, only the ReAct LangGraph is fully supported.
* `tools` (dictionary; optional): tool configuration
  * `searcher` (optional; defaults to "brightdata"): specification of SERP API provider (currently only one option: "brightdata")
  * `scraper` (optional; defaults to "brightdata:combo"): specification of web scraper (see docs below)
* `structured_output_format' (dictionary, optional): Lets you specify the shape of the response.
 * Each key is a property name.
 * Each value is a dictionary that defines the property type and an optional description.
 * If omitted, the response will be plain text.
* Use the following optional input parameters if you want the app to run asynchronously:
  * `id` (string): this will typically be a run ID used to correctly place the response **asynchronously on webhook**
  * `webhook_url` (string): webhook URL for receiving the asynchronous response in `{"id": "...", "response": {}}` format.
```aiignore
# structured_output_format
# 'items' field is required when the property type is "array"
{
  <PROPERTY_NAME>: {
    "type": <TYPE>, # types: string, array, boolean, number
    "items": {"type": <TYPE>},
    "description": <DESCRIPTION>
  }
}

#Example:
{
    "vat_id": {
        "type": "string",
        "description": "VAT number, also known as (sales) tax number."
    },
    "vat_associated_name": {
        "type": "string",
        "description": "Company/subsidiary name associated with the VAT_ID."
    }
}
```

### Brightdata scraper
Brightdata's Web Unlocker - tool options (format `<provider>:<specification>`)
1. `brightdata:combo`
   - First, a simple HTTP `GET` request is attempted (without a proxy).
   - If that  fails, Brightdata's Web Unlocker is used.

### Apify scraper
Apify's RAG Web Browser - tool options (format `<provider>:<specification>`:)
1. `apify:raw-http`
   - no JavaScript rendering — uses basic HTTP `GET` functionality with proxy etc.
   - fast & cheap
2. `apify:browser-playwright`
   - headless browser for JavaScript rendering etc.
   - pricey and slower, but gets more content
3. `apify:combo`: the agent is initialized with two scraping tools (different settings)
   - `raw-http`: this tool should be always used for the first scraping attempt
   - `browser-playwright` (with `check_past_scrapes` option off): the agent is instructed to use this tool if the `raw-http` didn't return a satisfactory response (likely caused by a JavaScript-heavy site)

To function in **standby mode**, when the actor run is reused by multiple requests instead of being initialized each time, implement the RAG Web Browser with an HTTP request. To control some actor settings such as RAM memory, it's best to use Apify's UI to create a Task where a configuration shared among runs is permanently stored.

#### Apify Datasets
Apify uses the concept of datasets. Each actor's run (multiple scrapings within the same run) stores scraped results in a dataset. By default, the agent's tool first checks the past few runs to see whether the requested URL has already been scraped. If so, it will return that result. If not, it scrapes according to its configuration.

## To Dos
Test open source models:
https://huggingface.co/deepseek-ai/DeepSeek-R1-Distill-Qwen-32B
https://huggingface.co/meta-llama/Llama-3.3-70B-Instruct
https://huggingface.co/meta-llama/Llama-3.1-8B-Instruct
https://huggingface.co/mistralai/Mixtral-8x7B-Instruct-v0.1
https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.3
