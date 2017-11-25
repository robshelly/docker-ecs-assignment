FROM python:2

RUN pip install pika

ADD send.py /code/send.py
ARG rabbitmq_ip
RUN sed -i s/rabbitmq_ip/$rabbitmq_ip/g /code/send.py