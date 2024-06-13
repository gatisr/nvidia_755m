<https://github.com/AbdBarho/stable-diffusion-webui-docker/wiki/Setup>

RUN

```bash
docker compose --profile download up --build
# wait until its done, then:
docker compose --profile auto up --build
# where [ui] is one of: invoke | auto | auto-cpu | comfy | comfy-cpu
```

if you don't know which ui to choose, invoke or auto are a good start.

Then access from <http://localhost:7860/>

```bash
pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
```
