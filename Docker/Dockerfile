FROM python:3.10-slim

RUN apt-get update && apt-get install -y tar && rm -rf /var/lib/apt/lists/*

# структура как на VM
RUN mkdir -p /opt/restoringvalues/{releases,current,run}

WORKDIR /opt/restoringvalues

# Артефакты (Jenkins положит в workspace и передаст в docker build как build context)
COPY deploy_art/app-restoringvalues.tgz /tmp/app-restoringvalues.tgz
RUN mkdir -p /opt/restoringvalues/current \
 && tar -xzf /tmp/app-restoringvalues.tgz -C /opt/restoringvalues/current

COPY deploy_art/dist/restoringvalues-0.1.0-py3-none-any.whl /tmp/app.whl
RUN pip install --no-cache-dir /tmp/app.whl

# стартовый скрипт
COPY docker/start.sh /start.sh
RUN chmod +x /start.sh

EXPOSE 8092 8093 8094 8095

CMD ["/start.sh"]
