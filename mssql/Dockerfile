FROM microsoft/mssql-server-linux:latest

COPY ./import-data.sh /
COPY ./entrypoint.sh /
COPY ./Products.csv /
COPY ./setup.sql /

RUN chmod +x /import-data.sh

EXPOSE 1433

CMD /bin/bash ./entrypoint.sh
