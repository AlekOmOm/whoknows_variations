.PHONY: init run test

init:
	PYTHONPATH=backend python2 -c "from app import init_db; init_db()"

docker-run:
	docker compose up --build

docker-test:
	docker compose stop whoknows_flask 
	sleep 130
	docker compose start whoknows_flask

happy:
	@docker compose ps | grep "alertmanager-1" | grep "Up"
	@docker compose ps | grep "alertmanager-2" | grep "Up"
	@docker compose ps | grep "prometheus" | grep "Up"
	@docker compose ps | grep "grafana" | grep "Up"
	@echo "Prometheus, Alertmanager-1, Alertmanager-2, and Grafana are ready"


unhappy:
	curl "http://localhost:8080/?q=dsa"
	sleep 2 

run:
	python2 ./backend/app.py

test: 
	PYTHONPATH=backend python2 ./backend/app_tests.py


help:
	@echo "Usage: make <target>"
	@echo "Targets:"
	@echo "  init - Initialize the database"
	@echo "  docker-run - Run the Docker containers"
	@echo "  docker-test - Test the Docker containers"
	@echo "  run - Run the Python application"
	@echo "  test - Run the Python tests"
	@echo "  help - Show this help message"