# ChatPDF

#Install required packages:

pip install Flask openai pdfminer.six

#Create a Flask web application:

from flask import Flask, render_template, request, redirect, url_for, flash
import openai
from pdfminer.high_level import extract_text
import os

app = Flask(__name__)
app.secret_key = 'your_secret_key'

# Configure OpenAI API
openai.api_key = 'your_openai_api_key'

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/upload', methods=['POST'])
def upload():
    if 'file' not in request.files:
        flash('No file part')
        return redirect(request.url)

    file = request.files['file']

    if file.filename == '':
        flash('No selected file')
        return redirect(request.url)

    if file:
        # Save the uploaded PDF file
        pdf_path = os.path.join('uploads', file.filename)
        file.save(pdf_path)

        # Extract text from the PDF
        pdf_text = extract_text(pdf_path)

        # Process user's question and get an answer from GPT-3
        user_question = request.form['question']
        response = openai.Completion.create(
            engine="text-davinci-002",
            prompt=f"Question: {user_question}\nContext: {pdf_text}",
            max_tokens=150
        )

        answer = response.choices[0].text

        return render_template('result.html', pdf_text=pdf_text, user_question=user_question, answer=answer)

if __name__ == '__main__':
    app.run(debug=True)
    
