FROM golang as builder

RUN mkdir /app

ADD https://raw.githubusercontent.com/adterskov/geekbrains-conteinerization/master/homework/2.docker/golang/go.mod /app
ADD https://raw.githubusercontent.com/adterskov/geekbrains-conteinerization/master/homework/2.docker/golang/go.sum /app
ADD https://raw.githubusercontent.com/adterskov/geekbrains-conteinerization/master/homework/2.docker/golang/main.go /app

WORKDIR /app
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o server .


FROM scratch

COPY --from=builder /app/server /

EXPOSE 8080
CMD ["/server"]