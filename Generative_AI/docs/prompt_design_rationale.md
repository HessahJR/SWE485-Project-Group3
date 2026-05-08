# Prompt Design Rationale

This document explains the rationale behind the prompt templates used in the Generative AI component of the loan approval system.
---

# Template 1: Loan Eligibility Evaluator

## Intended Use Case
This template is designed for bank employees who need to make quick decisions and provide simple explanations when customers inquire about loan eligibility. It helps determine whether a customer can obtain a loan and provides a brief justification that can be easily communicated to the customer.

## Design Rationale
 A simple and direct prompt was selected to ensure concise and clear outputs. This approach minimizes unnecessary information and improves response efficiency, making it suitable for users who require only the final decision along with its main justification.

## Thought Process Behind the Template
The main idea behind this template is to simplify the decision explanation process. The primary goal is to deliver a clear decision to bank employees, particularly in situations where a customer inquires about the reason behind a loan approval or rejection. The template is designed to be consistent and straightforward. The prompt is structured by first providing the Gemini model with the applicant's financial information followed by the machine learning decision, which helps the model understand the full context before generating a response that clearly states whether the loan was approved 
or rejected along with the most important reason behind that decision.

## How Domain Knowledge Influenced the Design
Loan approval or rejection decisions are strongly driven by financial indicators such as annual income, requested loan amount, credit profile, loan-to-income ratio, total assets, and asset-to-loan ratio. For this reason, these indicators were  included in the prompt to provide the model 
with the most relevant context for generating a meaningful explanation.

The role assignment "You are a bank loan officer" was also  added, as bank officers are responsible for communicating loan decisions to 
customers in a professional and straightforward manner. This ensures that the generated response aligns with language expected in a real 
banking context.

Finally, the decision to limit the response to 1-2 sentences was influenced by banking practice, where employees often need quick and direct justifications 
to communicate to customers.

## Prompt Structure
```python
def template_1(features: dict, prediction: str) -> str:
    prompt = f"""
You are a bank loan officer.

A loan application has been evaluated by a machine learning model.

Applicant Information:
- Number of Dependents: {features['no_of_dependents']}
- Education Level: {'Graduate' if features['education'] == 1 else 'Not Graduate'}
- Self Employment Status: {'Yes' if features['self_employed'] == 1 else 'No'}
- Annual Income: {features['income_annum']:,}
- Loan Amount Requested: {features['loan_amount']:,}
- Residential Assets Value: {features['residential_assets_value']:,}
- Commercial Assets Value: {features['commercial_assets_value']:,}
- Luxury Assets Value: {features['luxury_assets_value']:,}
- Bank Asset Value: {features['bank_asset_value']:,}
- Loan Term: {features['loan_term']} years
- CIBIL Category: {'Low' if features['cibil_category'] == 0 else 'Medium' if features['cibil_category'] == 1 else 'High'}
- Loan to Income Ratio: {features['loan_to_income_ratio']:.2f}
- Total Assets Value: {features['total_assets']:,}
- Asset to Loan Ratio: {features['asset_to_loan_ratio']:.2f}

Model Decision: {prediction}

Provide a brief summary of the loan decision and the most important reason behind it in 1-2 sentences. The explanation should be professional and suitable for bank employees.
"""
    return call_gemini(prompt)
```

## Example Input/Output
### input Information:

- Number of Dependents: 1
- Education: Graduate
- Self Employed: No
- Annual Income: 9,600,000
- Loan Amount Requested: 29,900,000
- Loan Term: 12 years
- residential_assets_value: 2400000
- commercial_assets_value : 17600000
- luxury_assets_value : 22700000
- bank_asset_value : 8000000,
- CIBIL Category: High
- Loan to Income Ratio: 3.11
- Total Assets Value: 50,700,000
- Asset to Loan Ratio: 1.69

### Model Prediction:
Approved

### Output: 
The loan application has been approved based on the applicant’s high CIBIL category and strong total asset value of 50,700,000, which significantly exceeds the requested loan amount. These factors indicate strong financial stability and low credit risk.

## Assumptions & Limitations
### Assumptions:
- The ML model (Random Forest) prediction is assumed to be correct and reliable.
- The applicant's provided financial data is accurate and complete.
### Limitations:
- Includes the Random Forest model prediction as part of the input, meaning the quality of the explanation depends on the accuracy of the prediction.
- Does not include clustering results, so no group-level personalization is applied.
- Provides only a brief 1-2 sentence explanation without detailed reasoning.
- May not handle complex or borderline cases accurately, as the prompt is designed for straightforward decisions.
- Response length is intentionally limited, which may lack depth for cases requiring more nuanced explanations.

## Lessons Learned During Prompt Testing
During prompt testing, the template showed that limiting the response to 1-2 sentences effectively reduces irrelevant output and keeps the explanation 
focused. However, this also restricts the depth of explanation, particularly for borderline cases where multiple factors contribute to the decision.

Testing also showed that clear role assignment is important. By instructing the model to act as a bank loan officer, the responses maintained a 
professional and consistent tone across different test cases.

Another lesson learned is that including financial indicators such as CIBIL category, loan-to-income ratio, and asset-to-loan ratio improves the relevance of the generated explanation. However, the model tends to prioritize these indicators over other features such as income or employment status, 
which may result in explanations that overlook other contributing factors.


---

# Template 2: 

## Intended Use Case
This template is designed for bank employees, financial analysts, or loan reviewers who need a more detailed explanation of the machine learning decision. Unlike Template 1, which only provides a short summary, this template explains the main financial strengths, weaknesses, and overall risk level of the applicant.

## Design Rationale
A more structured and descriptive prompt was selected to generate deeper analytical explanations. This approach helps stakeholders understand not only the final outcome, but also the reasoning behind it based on the applicant’s financial profile. This template is useful when a short answer is not enough and more context is needed for review or internal discussion.


## Thought Process Behind the Template
Template 2 was designed to provide more detailed financial explanations compared to the brief summaries generated by Template 1. The goal was to create responses that resemble professional financial assessments by analyzing multiple financial indicators together instead of focusing on a single reason.

The template was structured to discuss both strengths and weaknesses in the applicant’s profile while maintaining a professional and analytical tone suitable for bank employees and financial analysts. Instructions such as “Do not give advice or recommendations” were included to keep the output focused on explanation rather than guidance.

## How Domain Knowledge Influenced the Design
Domain knowledge influenced the selection of the financial indicators included in the prompt, such as annual income, asset-to-loan ratio, loan-to-income ratio, total assets, and CIBIL category. These indicators are commonly used in real loan evaluation processes to assess repayment ability and financial risk.

The prompt was designed to reflect how financial professionals interpret these values. For example, a high asset-to-loan ratio suggests stronger collateral coverage, while a high loan-to-income ratio indicates higher repayment risk. Including these indicators helped make the generated explanations more realistic and financially relevant.

## Prompt Structure

```python
def template_2(features: dict, prediction: str) -> str:
    prompt = f"""
You are a senior financial analyst reviewing a loan application evaluated by a machine learning model.

Applicant Information:
- Number of Dependents: {features['no_of_dependents']}
- Education Level: {'Graduate' if features['education'] == 1 else 'Not Graduate'}
- Self Employment Status: {'Yes' if features['self_employed'] == 1 else 'No'}
- Annual Income: {features['income_annum']:,}
- Loan Amount Requested: {features['loan_amount']:,}
- Residential Assets Value: {features['residential_assets_value']:,}
- Commercial Assets Value: {features['commercial_assets_value']:,}
- Luxury Assets Value: {features['luxury_assets_value']:,}
- Bank Asset Value: {features['bank_asset_value']:,}
- Loan Term: {features['loan_term']} years
- CIBIL Category: {'Low' if features['cibil_category'] == 0 else 'Medium' if features['cibil_category'] == 1 else 'High'}
- Loan to Income Ratio: {features['loan_to_income_ratio']:.2f}
- Total Assets Value: {features['total_assets']:,}
- Asset to Loan Ratio: {features['asset_to_loan_ratio']:.2f}

Model Decision: {prediction}

Provide a detailed explanation of the decision in one short paragraph.
Your response should:
1. Clearly state whether the applicant appears financially strong or risky.
2. Mention the most important strengths and weaknesses in the profile.
3. Explain why these factors support the model decision.
4. Keep the explanation professional, clear, and understandable.
Do not give advice or recommendations. Focus only on analysis.
"""
    return call_gemini(prompt)
```
## Example Input/Output

### input Information:
- Number of Dependents: 1
- Education: Graduate
- Self Employed: No
- Annual Income: 9,600,000
- Loan Amount Requested: 29,900,000
- Loan Term: 12 years
- residential_assets_value: 2400000
- commercial_assets_value : 17600000
- luxury_assets_value : 22700000
- bank_asset_value : 8000000,
- CIBIL Category: High
- Loan to Income Ratio: 3.11
- Total Assets Value: 50,700,000
- Asset to Loan Ratio: 1.69

### Model Prediction:
Approved

### output:
The applicant appears financially very strong, exhibiting robust repayment capacity and substantial collateral, which strongly supports the model’s approval decision. Key strengths include the high annual income of 9,600,000, excellent CIBIL category, and significant total assets valued at 50,700,000, resulting in a strong asset-to-loan ratio of 1.69. Although the requested loan amount is relatively large, the applicant’s strong financial indicators and manageable loan-to-income ratio suggest low repayment risk and good overall financial stability.

## Assumptions & Limitations

### Assumptions:
- The machine learning prediction is assumed to be correct and reliable.
- The applicant’s financial information is complete and accurate.
- The listed financial indicators are sufficient for generating a meaningful explanation.

### Limitations:
- The output depends on the quality of the model prediction.
- The explanation is more detailed than Template 1, but it still does not provide personalized next steps.
- The template focuses on financial interpretation only and does not account for external real-world factors not included in the dataset.
- The model may emphasize some indicators more than others, even when multiple factors contribute to the decision.

## Lessons Learned During Prompt Testing
During testing, it was observed that adding more structured instructions improved the consistency and detail of the generated explanations. The model became better at discussing multiple financial factors and aligning the explanation with the prediction outcome.
Another important observation was the trade-off between detail and readability. While Template 2 produced richer and more informative responses, the explanations became more technical and harder for non-expert users to understand. The testing process also showed that small changes in prompt wording could significantly affect the style and focus of the generated output.

---

# Template 3: Loan Improvement Advisor

## Intended Use Case

This template is designed for bank employees who need to explain loan rejection or borderline decisions and provide practical recommendations that can be communicated to customers. It supports employees in delivering clear, concise justifications along with actionable suggestions to help customers improve their chances of loan approval in the future.

## Design Rationale

A structured prompt was selected to ensure that responses are clear, concise, and actionable. Unlike simple explanation templates, this approach supports employees in providing constructive feedback by including practical recommendations. This improves the overall usefulness of the response and makes it more suitable for scenarios where customers expect guidance on how to improve their chances of loan approval.

## Thought Process Behind the Template

The main idea behind this template is to move beyond simply reporting whether a loan application is approved or rejected. In real banking scenarios, employees often need to explain unfavorable decisions in a professional and helpful way. Therefore, this template was designed to support both decision explanation and improvement guidance.

The template first provides the Gemini model with the applicant’s financial information, the machine learning decision, and the applicant profile group. This structure helps the model understand the context before generating the response. The prompt then separates the expected behavior based on the decision outcome. If the application is approved, the response should remain brief and focus on the strongest approval factor. If the application is rejected, the response should include a short explanation and three practical recommendations that the employee can communicate to the customer.

## How Domain Knowledge Influenced the Design

Loan approval decisions are strongly affected by financial indicators such as annual income, requested loan amount, credit profile, loan-to-income ratio, total assets, and asset-to-loan ratio. These indicators were included in the prompt because they provide important context for explaining approval or rejection decisions.

The template also includes the applicant profile group from the clustering component. This adds a higher-level view of the applicant’s financial segment, such as whether the applicant belongs to a high-asset group, a modest-income group, or a high-loan-burden group. Including this information helps the generated response become more personalized and aligned with the applicant’s situation.

The recommendation part is also influenced by banking domain logic. For example, if an applicant has a high loan-to-income ratio, a suitable recommendation may be to request a lower loan amount. If the asset-to-loan ratio is low, the response may recommend increasing savings or strengthening asset support. This ensures that the generated advice is connected to the applicant’s financial profile rather than being generic.

## Prompt Structure

```python
def template_3(features: dict, prediction: str, cluster_profile: str) -> str:
    prompt = f"""
You are a bank loan officer.

A loan application has been evaluated by a machine learning model.

Applicant Information:
- Number of Dependents: {features['no_of_dependents']}
- Education Level: {'Graduate' if features['education'] == 1 else 'Not Graduate'}
- Self Employment Status: {'Yes' if features['self_employed'] == 1 else 'No'}
- Annual Income: {features['income_annum']:,}
- Loan Amount Requested: {features['loan_amount']:,}
- Residential Assets Value: {features['residential_assets_value']:,}
- Commercial Assets Value: {features['commercial_assets_value']:,}
- Luxury Assets Value: {features['luxury_assets_value']:,}
- Bank Asset Value: {features['bank_asset_value']:,}
- Loan Term: {features['loan_term']} years
- CIBIL Category: {'Low' if features['cibil_category'] == 0 else 'Medium' if features['cibil_category'] == 1 else 'High'}
- Loan to Income Ratio: {features['loan_to_income_ratio']:.2f}
- Total Assets Value: {features['total_assets']:,}
- Asset to Loan Ratio: {features['asset_to_loan_ratio']:.2f}

Applicant Profile Group: {cluster_profile}

Model Decision: {prediction}

If the decision is approved, provide a brief summary of the loan decision and the most important reason behind it in 1-2 sentences.

If the decision is rejected, provide:
1. A brief explanation of the rejection decision.
2. Three short and practical recommendations that the employee can communicate to the customer to improve the chances of loan approval in the future.

Keep the response clear, concise, and professional.
"""
    return call_gemini(prompt)
```
## Example Input/Output

### Input Information
- Number of Dependents: 4
- Education: Graduate
- Self Employed: Yes
- Annual Income: 2,000,000
- Loan Amount Requested: 25,000,000
- Loan Term: 20 years
- Residential Assets Value: 500,000
- Commercial Assets Value: 1,000,000
- Luxury Assets Value: 800,000
- Bank Asset Value: 300,000
- CIBIL Category: Low
- Loan to Income Ratio: 12.50
- Total Assets Value: 2,600,000
- Asset to Loan Ratio: 0.10

### Applicant Profile Segment
High-leverage / stretched applicant.

### Model Prediction
Rejected

### Output
The loan application has been rejected due to the very high loan-to-income ratio and low asset-to-loan ratio, which indicate a high financial risk. To improve approval chances, the applicant should consider requesting a lower loan amount, increasing total assets or savings, and improving their credit profile over time.

---

## Assumptions & Limitations

### Assumptions
- The ML model (Random Forest) prediction is assumed to be correct and reliable.
- The applicant's provided financial data is accurate and complete.
- The cluster profile accurately represents the applicant’s financial segment.

### Limitations
- Includes the Random Forest model prediction as part of the input, meaning the quality of the explanation depends on the accuracy of the prediction.
- May not fully capture complex or borderline cases accurately.
- Recommendations are general and based on observed financial indicators, not personalized financial planning.
- The response is intentionally concise, which may limit deeper financial analysis.

---

## Lessons Learned During Prompt Testing

During prompt testing, the template showed that adding improvement recommendations makes the output more useful for rejection cases compared to templates that only explain the decision. The responses became more practical because they helped employees communicate not only why the application was rejected, but also what the customer could improve in the future.

Testing also showed that clear conditional instructions are important. By separating the approved and rejected cases inside the prompt, the model was more likely to generate the correct type of response for each decision. Approved cases remained short and focused, while rejected cases included explanation and advice.

Another lesson learned is that including financial indicators and the applicant profile group improves personalization. The model can refer to specific risk factors, such as high loan-to-income ratio or low asset-to-loan ratio, instead of producing vague recommendations. However, the prompt must remain concise to avoid generating overly long or overly technical responses.

---

# Template 4: Customer-Friendly Loan Explanation

## Intended Use Case

This template is designed for everyday customers and non-expert clients who interact with the bank through self-service channels (mobile apps, customer portals, or front-desk interactions). It is intended to explain loan approval decisions in a clear, friendly, and easy-to-understand way without requiring financial knowledge. Unlike previous templates that target employees or analysts, this template focuses on accessibility, emotional tone, and personalization so that the client feels respected and informed regardless of the outcome.

## Design Rationale

A conversational and supportive prompt style was selected to ensure that responses are simple, human-like, and easy to understand. The goal is to translate technical financial indicators into everyday language so that non-expert clients can clearly understand the reason behind the decision. The prompt also emphasizes tone and emotional clarity, especially in rejection cases, to maintain a positive client experience. This makes the template more suitable for direct customer communication compared to technical or analytical templates.

## Thought Process Behind the Template

Template 4 was designed to bridge the gap between technical model outputs and real-world client communication. While earlier templates focused on accuracy and financial reasoning, this template prioritizes how the message is perceived by the client.

The prompt first provides the model with detailed applicant financial data, the model decision, and the cluster-based profile group. This ensures that the model has full context before generating the response. The instructions then guide the model to simplify the explanation, avoid technical terms, and present the decision in a warm and supportive tone.

Special attention was given to rejection cases. Instead of presenting a purely analytical explanation, the template encourages a gentle and encouraging message that helps maintain trust and avoids discouraging the client.

## How Domain Knowledge Influenced the Design

Loan decisions are typically based on financial indicators such as income, credit history, loan size, and asset values. However, these indicators are often expressed using technical terms (e.g., loan-to-income ratio, asset-to-loan ratio) that are difficult for non-expert clients to understand.

Domain knowledge influenced the decision to translate these technical metrics into plain language concepts such as:
- “ability to repay the loan”
- “overall savings and assets”
- “financial stability”

In addition, the inclusion of the cluster profile (e.g., modest-income applicant, affluent applicant) reflects how banks segment clients into financial groups. This allows the explanation to feel more personalized and relevant to the client’s situation.

Finally, the tone of communication was influenced by real banking practices, where customer-facing communication must remain polite, respectful, and supportive, even when delivering negative decisions.

## Prompt Structure

```python
def template_4(features: dict, prediction: str, cluster_profile: str) -> str:
    prompt = f"""
You are a friendly and supportive bank assistant speaking directly to a customer.

The customer's loan application has been reviewed by an automated system. Your job is to explain the result in simple, everyday language, as if you were sitting across from the customer in a branch — warm, respectful, and easy to understand.

Customer Information:
- Number of Dependents: {features['no_of_dependents']}
- Education Level: {'Graduate' if features['education'] == 1 else 'Not Graduate'}
- Self Employment Status: {'Yes' if features['self_employed'] == 1 else 'No'}
- Annual Income: {features['income_annum']:,}
- Loan Amount Requested: {features['loan_amount']:,}
- Residential Assets Value: {features['residential_assets_value']:,}
- Commercial Assets Value: {features['commercial_assets_value']:,}
- Luxury Assets Value: {features['luxury_assets_value']:,}
- Bank Asset Value: {features['bank_asset_value']:,}
- Loan Term: {features['loan_term']} years
- Credit History Strength: {'Weak' if features['cibil_category'] == 0 else 'Average' if features['cibil_category'] == 1 else 'Strong'}
- Loan to Income Ratio: {features['loan_to_income_ratio']:.2f}
- Total Assets Value: {features['total_assets']:,}
- Asset to Loan Ratio: {features['asset_to_loan_ratio']:.2f}

Customer Profile Group: {cluster_profile}

System Decision: {prediction}

Write a short, friendly response of 3 to 4 sentences that:
1. Greets the customer warmly and shares the decision in plain language.
2. Explains the main reason behind the decision using everyday words (avoid technical terms).
3. Personalizes the message using the customer profile group.
4. Ends with a supportive closing message.

Keep the tone warm, respectful, and easy to understand.
"""
    return call_gemini(prompt)
```

## Example Input/Output

### Input Information
- Number of Dependents: 1
- Education: Graduate
- Self Employed: No
- Annual Income: 9,600,000
- Loan Amount Requested: 29,900,000
- Loan Term: 12 years
- Residential Assets Value: 2,400,000
- Commercial Assets Value: 17,600,000
- Luxury Assets Value: 22,700,000
- Bank Asset Value: 8,000,000
- Credit History Strength: Strong
- Loan to Income Ratio: 3.11
- Total Assets Value: 50,700,000
- Asset to Loan Ratio: 1.69

### Applicant Profile Segment
Affluent / large-loan applicant.

### Model Prediction
Approved

### Output
Hello there! I have some fantastic news regarding your loan application – it's been approved! Our automated system highlighted your excellent credit history and the healthy value of your assets, showing us you have a very strong financial foundation. These factors were key in approving such a significant loan for a valued client like yourself. Congratulations, and we're truly happy to help you move forward!

---

## Assumptions & Limitations

### Assumptions
- The machine learning prediction is assumed to be correct and reliable.
- The applicant's financial information is complete and accurate.
- The cluster profile accurately represents the client’s overall financial segment and is meaningful enough to support personalization.
- The client is a non-expert reader who benefits more from simple and accessible language than from technical detail.

### Limitations
- The output depends on the quality of the model prediction; an incorrect prediction will still be explained in a clear but potentially misleading way.
- Simplifying technical indicators reduces analytical precision, making this template unsuitable for internal financial analysis.
- Personalization is limited to the cluster profile and does not include deeper individual context such as customer goals or history with the bank.
- The friendly tone used in rejection cases may be perceived as too soft in more formal communication settings.
- The responses are generated in English only and may require localization for multilingual client bases.

---

## Lessons Learned During Prompt Testing

During testing, Template 4 demonstrated strong performance in delivering clear and user-friendly explanations across all three test cases. The responses consistently reflected the model’s decision while maintaining a warm and supportive tone, making them easy for non-expert clients to understand.

In approved cases, the explanation highlights positive financial factors such as strong credit history and high asset value, reinforcing confidence in the decision. In rejected cases, the template avoids technical language and instead explains the outcome using simple reasoning, while providing gentle encouragement to maintain a positive client experience.

Another key observation is that using the client’s profile group improves personalization, making the response feel more relevant rather than generic. However, this comes with a trade-off, as the explanations prioritize clarity and tone over detailed financial reasoning.

Overall, Template 4 is highly effective for customer-facing communication, where readability, tone, and personalization are more important than technical depth.


---
## References to Prompt Engineering Best Practices

The prompt templates were designed based on common prompt engineering practices.

For this project, the prompts followed these best practices by:

- Defining the model role as a bank loan officer.
- Providing structured applicant information.
- Including the model prediction as context.
- Specifying different output behavior for approved and rejected cases.
- Keeping the response clear, concise, and professional.
- Testing the prompt using strong, weak, and borderline applicant profiles.

### IEEE References

[1] OpenAI, “Best practices for prompt engineering with the OpenAI API,” OpenAI Help Center. Accessed: May 1, 2026. [Online]. Available: https://help.openai.com/en/articles/6654000-best-practices-for-prompt-engineering-with-the-openai-api

[2] Google AI for Developers, “Prompt design strategies,” Gemini API Documentation. Accessed: May 1, 2026. [Online]. Available: https://ai.google.dev/gemini-api/docs/prompting-strategies

[3] S. Schulhoff et al., “The Prompt Report: A Systematic Survey of Prompting Techniques,” arXiv preprint arXiv:2406.06608, 2024. [Online]. Available: https://arxiv.org/abs/2406.06608
