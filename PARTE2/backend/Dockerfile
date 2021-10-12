FROM python:3.7

ENV PYTHONUNBUFFERED 1

WORKDIR /code

COPY requirements.txt .

RUN apt-get update \
    && apt-get install -y graphviz-dev \
    && pip install --upgrade pip \
    && pip install psycopg2-binary \
    && pip3 install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]