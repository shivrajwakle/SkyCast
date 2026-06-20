# SkyCast

import sys
import requests
from PyQt5.QtWidgets import QApplication, QPushButton, QVBoxLayout, QWidget, QLabel, QLineEdit
from PyQt5.QtCore import Qt


class WeatherApp(QWidget):
    def __init__(self):
        super().__init__()

        self.city_label = QLabel("Enter City Name:", self)
        self.city_input = QLineEdit(self)
        self.get_weather_button = QPushButton("Get Weather", self)

        self.temperature_label = QLabel(self)
        self.emoji_label = QLabel(self)
        self.description_Label = QLabel(self)

        self.initUI()

        self.get_weather_button.clicked.connect(self.get_weather)

    def initUI(self):
        self.setWindowTitle("Weather App")

        vbox = QVBoxLayout()

        vbox.addWidget(self.city_label)
        vbox.addWidget(self.city_input)
        vbox.addWidget(self.get_weather_button)
        vbox.addWidget(self.temperature_label)
        vbox.addWidget(self.emoji_label)
        vbox.addWidget(self.description_Label)

        self.setLayout(vbox)

        self.city_label.setAlignment(Qt.AlignCenter)
        self.city_input.setAlignment(Qt.AlignCenter)
        self.temperature_label.setAlignment(Qt.AlignCenter)
        self.emoji_label.setAlignment(Qt.AlignCenter)
        self.description_Label.setAlignment(Qt.AlignCenter)

        self.city_label.setObjectName("city_label")
        self.city_input.setObjectName("city_input")
        self.get_weather_button.setObjectName("get_weather_button")
        self.temperature_label.setObjectName("temperature_label")
        self.emoji_label.setObjectName("emoji_label")
        self.description_Label.setObjectName("description_label")

        self.setStyleSheet("""
            QLabel, QPushButton{
                font-family: Calibri;
            }

            QLabel#city_label{
                font-size: 40px;
                font-style: italic;
            }

            QLineEdit#city_input{
                font-size: 40px;
            }

            QPushButton#get_weather_button{
                font-size: 30px;
                font-weight: bold;
            }

            QLabel#temperature_label{
                font-size: 70px;
            }

            QLabel#emoji_label{
                font-size: 100px;
                font-family: "Segoe UI Emoji";
            }

            QLabel#description_label{
                font-size: 50px;
            }
        """)

    def get_weather(self):
        api_key = "9d6ea6ac336c9bd41cf46da141402476"
        city = self.city_input.text().strip()

        if not city:
            self.display_error("Please enter a city name")
            return

        url = f"https://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}"

        try:
            response = requests.get(url, timeout=10)
            response.raise_for_status()

            data = response.json()

            if data["cod"] == 200:
                self.display_weather(data)

        except requests.exceptions.HTTPError as http_error:

            match response.status_code:

                case 400:
                    self.display_error("Bad Request\nPlease check your input")

                case 401:
                    self.display_error("Unauthorized\nPlease check your API key")

                case 403:
                    self.display_error("Forbidden\nAccess denied")

                case 404:
                    self.display_error("City Not Found")

                case 500:
                    self.display_error("Internal Server Error")

                case 502:
                    self.display_error("Bad Gateway")

                case 503:
                    self.display_error("Service Unavailable")

                case 504:
                    self.display_error("Gateway Timeout")

                case _:
                    self.display_error(f"HTTP Error:\n{http_error}")

        except requests.exceptions.ConnectionError:
            self.display_error("Connection Error\nCheck internet")

        except requests.exceptions.Timeout:
            self.display_error("Request Timed Out")

        except requests.exceptions.TooManyRedirects:
            self.display_error("Too Many Redirects")

        except requests.exceptions.RequestException as req_error:
            self.display_error(f"Request Error:\n{req_error}")

        except Exception as e:
            self.display_error(f"Unexpected Error:\n{e}")

    def display_error(self, message):
        self.temperature_label.setText(message)
        self.temperature_label.setStyleSheet("font-size: 30px;")

        self.emoji_label.clear()
        self.description_Label.clear()

    def display_weather(self, data):
        temperature_k = data["main"]["temp"]
        temperature_c = temperature_k - 273.15
        weather_id = data["weather"][0]["id"]
        weather_description = data["weather"][0]["description"]

        self.temperature_label.setStyleSheet("font-size: 70px;")
        self.temperature_label.setText(f"{temperature_c:.1f} °C")
        self.emoji_label.setText(self.get_weather_emoji(weather_id))
        self.description_Label.setText(weather_description.title())

    @staticmethod
    def get_weather_emoji(weather_id):

        if 200 <= weather_id <= 232:
          return "⛈️"

        elif 300 <= weather_id <= 321:
            return "🌦️"

        elif 500 <= weather_id <= 531:
            return "🌧️"

        elif 600 <= weather_id <= 622:
            return "❄️"

        elif 701 <= weather_id <= 741:
            return "🌫️"

        elif weather_id == 762:
            return "🌋"

        elif weather_id == 771:
            return "💨"

        elif weather_id == 781:
            return "🌪️"

        elif weather_id == 800:
            return "☀️"

        elif 801 <= weather_id <= 804:
            return "☁️"

        else:
            return "❓"

if __name__ == "__main__":
    app = QApplication(sys.argv)

    weather_app = WeatherApp()
    weather_app.show()

    sys.exit(app.exec_())
