FROM python:3.9-alpine

RUN pip install Flask==1.1.2

ADD https://raw.githubusercontent.com/adterskov/geekbrains-conteinerization/master/homework/2.docker/python/app.py .

EXPOSE 8080

ENTRYPOINT python app.py