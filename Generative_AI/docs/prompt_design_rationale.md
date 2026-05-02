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

Provide a brief summary of the loan decision and the most important reason behind it in 1-2 sentences.
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
- Model Prediction: Approved

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


## Design Rationale


## Thought Process Behind the Template


## How Domain Knowledge Influenced the Design


## Prompt Structure


## Example Input/Output


## Assumptions & Limitations


## Lessons Learned During Prompt Testing

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

# Template 4: 

## Intended Use Case


## Design Rationale


## Thought Process Behind the Template


## How Domain Knowledge Influenced the Design


## Prompt Structure


## Example Input/Output


## Assumptions & Limitations


## Lessons Learned During Prompt Testing


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
