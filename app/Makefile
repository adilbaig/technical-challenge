AWS_ACCESS_KEY=123
AWS_SECRET_KEY=abc

.PHONY: build-image
build-image:
	packer build \
		-var "aws_access_key=$(AWS_ACCESS_KEY)" \
		-var "aws_secret_key=$(AWS_SECRET_KEY)" \
		../packer-ami.json

virtualenv:
	virtualenv venv

.PHONY: build
build: virtualenv
	. venv/bin/activate && pip install -Ur requirements.txt

.PHONY: run
run:
	./venv/bin/python app.py --port 8080 --monitor 8081
