# Chatbot for Insurance FAQs using C# and ML.NET

## 📌 Overview
This project is a **chatbot** that processes insurance-related questions using **C# and ML.NET**. The bot categorizes user questions (e.g., "Third-Party Insurance" or "Car Body Insurance") and retrieves the correct answer from a **SQL Server database**.

## 🔥 Features
- **NLP-based Question Classification** using ML.NET
- **SQL Server Database** for storing and retrieving FAQs
- **C# Integration** with ML.NET and SQL Server
- **Scalable Architecture** for future integrations (e.g., APIs, Telegram, WhatsApp)

---

## 🏗️ Project Structure
```
/InsuranceChatbot
│── /ModelTraining        # ML.NET model training scripts
│── /Database             # SQL scripts for table & data setup
│── /BotLogic             # C# chatbot implementation
│── README.md             # Project documentation
│── InsuranceChatbot.sln  # Solution file
```

---

## ⚙️ Step 1: Setting Up SQL Server Database
### **1. Create the FAQ Table**
```sql
CREATE TABLE InsuranceFAQs (
    Id INT IDENTITY PRIMARY KEY,
    Category NVARCHAR(100),
    Question NVARCHAR(MAX),
    Answer NVARCHAR(MAX)
);
```

### **2. Insert Sample Questions & Answers**
```sql
INSERT INTO InsuranceFAQs (Category, Question, Answer)
VALUES
('ThirdPartyInsurance', 'بیمه شخص ثالث چه خساراتی را پوشش می‌دهد؟', 'بیمه شخص ثالث خسارات مالی و جانی اشخاص ثالث را پوشش می‌دهد.'),
('CarBodyInsurance', 'آیا بیمه بدنه سرقت خودرو را پوشش می‌دهد؟', 'بله، در صورت داشتن پوشش سرقت، خسارت جبران خواهد شد.');
```

---

## 🧠 Step 2: Training ML.NET Model for Question Classification
### **1. Define the Training Pipeline**
```csharp
var pipeline = mlContext.Transforms.Text.FeaturizeText("Features", "Question")
    .Append(mlContext.Transforms.Conversion.MapValueToKey("Label", "Category"))
    .Append(mlContext.Transforms.Concatenate("Features"))
    .Append(mlContext.MulticlassClassification.Trainers.SdcaMaximumEntropy("Label", "Features"))
    .Append(mlContext.Transforms.Conversion.MapKeyToValue("PredictedLabel"));
```

### **2. Train the Model & Save It**
```csharp
IDataView trainingData = mlContext.Data.LoadFromTextFile<QuestionData>("trainingData.csv", separatorChar: ',', hasHeader: true);
var model = pipeline.Fit(trainingData);
mlContext.Model.Save(model, trainingData.Schema, "model.zip");
```

---

## 🔗 Step 3: Connecting C# to SQL Server
### **1. Install SQL Client Library**
```sh
dotnet add package System.Data.SqlClient
```

### **2. Retrieve Answer from Database**
```csharp
public class DatabaseHelper
{
    private string connectionString = "Server=YOUR_SERVER;Database=YOUR_DATABASE;User Id=YOUR_USER;Password=YOUR_PASSWORD;";

    public string GetAnswer(string category)
    {
        string answer = "متأسفم، پاسخ مناسبی برای این سوال پیدا نشد.";
        using (SqlConnection connection = new SqlConnection(connectionString))
        {
            connection.Open();
            string query = "SELECT TOP 1 Answer FROM InsuranceFAQs WHERE Category = @Category";
            using (SqlCommand command = new SqlCommand(query, connection))
            {
                command.Parameters.AddWithValue("@Category", category);
                var result = command.ExecuteScalar();
                if (result != null)
                {
                    answer = result.ToString();
                }
            }
        }
        return answer;
    }
}
```

---

## 🎯 Step 4: Integrating Chatbot Logic
```csharp
var databaseHelper = new DatabaseHelper();
var predictedCategory = prediction.PredictedCategory;  // ML.NET output
string response = databaseHelper.GetAnswer(predictedCategory);
Console.WriteLine($"Chatbot: {response}");
```

---

## 🚧 Challenges & Future Improvements
### **1. Handling Persian NLP Issues**
- Stemming & Lemmatization (e.g., Hazm library)
- Spell Correction for user input errors
- Detecting similar questions

### **2. Enhancing Chatbot Abilities**
- Adding API integration for real-time updates
- Deploying the bot to Telegram / WhatsApp
- Expanding training data for better classification

---

## 🏁 Conclusion
This project demonstrates how to build a **Persian Insurance Chatbot** using **ML.NET & SQL Server**. You can further enhance it with **more training data, better NLP techniques, and API integration.** 🚀

---

## 📌 Next Steps
✅ Improve NLP processing with Persian libraries (e.g., Hazm)  
✅ Deploy as a web service or Telegram bot  
✅ Optimize SQL queries for faster response times  

Happy coding! 🚀😃

