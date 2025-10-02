from flask import Flask, request, jsonify
import pandas as pd

app = Flask(__name__)

# Load results from R script (CSV file)
rainfall_data = pd.read_csv("rainfall_results.csv")

@app.route('/')
def home():
    return "Rainfall & Drought API is running!"

@app.route('/monthly', methods=['GET'])
def monthly_data():
    # Return average monthly rainfall
    monthly = rainfall_data.groupby("Month")["Rainfall"].mean().to_dict()
    return jsonify(monthly)

@app.route('/annual', methods=['GET'])
def annual_data():
    # Return total rainfall per year
    annual = rainfall_data.groupby("Year")["Rainfall"].sum().to_dict()
    return jsonify(annual)

@app.route('/predict', methods=['POST'])
def predict():
    """
    Example endpoint: takes SPI value and predicts drought severity
    """
    data = request.get_json()
    spi = data.get("spi", 0)

    if spi <= -2:
        category = "Extreme Drought"
    elif spi <= -1.5:
        category = "Severe Drought"
    elif spi <= -1:
        category = "Moderate Drought"
    elif spi < 0:
        category = "Mild Drought"
    else:
        category = "No Drought"

    return jsonify({"SPI": spi, "Category": category})

if __name__ == "__main__":
    app.run(debug=True)
