# /********************************************************************************
# * Copyright (c) 2022 Contributors to the Eclipse Foundation
# *
# * See the NOTICE file(s) distributed with this work for additional
# * information regarding copyright ownership.
# *
# * This program and the accompanying materials are made available under the
# * terms of the Apache License 2.0 which is available at
# * http://www.apache.org/licenses/LICENSE-2.0
# *
# * SPDX-License-Identifier: Apache-2.0
# ********************************************************************************/

FROM python:3.9-slim-bullseye
WORKDIR /usr/app

COPY . .


RUN pip3 install -r requirements.txt

ENV LOG_LEVEL="INFO"
ENV DATABROKER_ADDRESS="localhost:55555"

CMD [ "python", "carsim_runner.py" ]
