FROM python:2

RUN pip install pika

ADD receive.py /code/receive.py
ARG rabbitmq_ip
RUN sed -i s/rabbitmq_ip/$rabbitmq_ip/g /code/receive.py