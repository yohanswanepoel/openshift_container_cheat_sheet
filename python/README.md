# Python examples

## Flask sample from base UBI
The sample build file uses a multi-stage build process. This speeds up building if there are python packages that require compilation.

```
FROM registry.redhat.io/ubi8/ubi-minimal as base

RUN microdnf install python38; microdnf clean all

RUN python3 -m venv /opt/venv

COPY ./requirements.txt .

ENV PATH="/opt/venv/bin:$PATH"

RUN pip install -r requirements.txt


FROM registry.access.redhat.com/ubi8/ubi-minimal as run-image

RUN microdnf install python38 tar

COPY --from=base /opt/venv /opt/venv

ENV PATH="/opt/venv/bin:$PATH"

ENV PORT=5000

EXPOSE 5000

WORKDIR /app

ADD . /app

RUN \
    chown -R 1001:0 /app && \
    chgrp -R 0 /app && \
    chmod -R g=u /app

USER 1001

ENTRYPOINT [ "python" ]

CMD ["app.py"]

```


Building the container
```bash
podman build -t test .
```

running the container
```bash
podman run -p 12000:5000 test

curl localhost:12000
```