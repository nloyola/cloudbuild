# pull official base image
FROM python:3.7.3-alpine
WORKDIR /django
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1
RUN pip install --upgrade pip
COPY ./requirements.txt /django/requirements.txt 
RUN pip install -r requirements.txt
COPY . /django/
CMD python ./backend/manage.py migrate && \
	 ./backend/manage.py runserver 0.0.0.0:$PORT 
