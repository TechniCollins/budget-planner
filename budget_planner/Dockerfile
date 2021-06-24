###########
# BUILDER #
###########

# pull official base image
FROM python:3.8.5-alpine as builder

# set work directory
WORKDIR /usr/src/budget_planner

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# install psycopg2 dependencies
RUN apk update \
    && apk add postgresql-dev gcc python3-dev musl-dev

# knox dependencies
RUN apk add libffi-dev openssl-dev cargo

# lint
RUN pip install --upgrade pip
RUN pip install flake8
COPY . .
RUN flake8 --ignore=E501 .

# install dependencies
COPY ./requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /usr/src/budget_planner/wheels -r requirements.txt


#########
# FINAL #
#########

# pull official base image
FROM python:3.8.5-alpine

# create directory for the app user
RUN mkdir -p /home/budget_planner

# create the app user
RUN addgroup -S budget_planner && adduser -S budget_planner -G budget_planner

# create the appropriate directories
ENV HOME=/home/budget_planner
ENV APP_HOME=/home/budget_planner/budget_planner
RUN mkdir $APP_HOME
RUN mkdir $APP_HOME/staticfiles
WORKDIR $APP_HOME

# install dependencies
RUN apk update && apk add libpq
COPY --from=builder /usr/src/budget_planner/wheels /wheels
COPY --from=builder /usr/src/budget_planner/requirements.txt .
RUN pip install --no-cache /wheels/*

# copy entrypoint.sh
COPY ./entrypoint.sh $APP_HOME

# copy project
COPY . $APP_HOME

# chown all the files to the app user
RUN chown -R budget_planner:budget_planner $APP_HOME

# change to the app user
USER budget_planner

# run entrypoint.sh
ENTRYPOINT ["/home/budget_planner/budget_planner/entrypoint.sh"]