# Appendix B: Title Filtering (Relevance Determination)

With the following prompt, irrelevant announcements will be removed:


You now need to analyze a given title. The analysis criteria are:

1. Determine whether it is related to equity investment, mergers & acquisitions, or asset restructuring.
   Notes:
2. The returned object must be a JSON structure.
3. The returned content must contain only the JSON structure.
4. The structure must contain only one field named "resutls". The value should be 1 for True, 0 for False.
5. You only need to determine Yes or No, and return 0 or 1 to indicate your judgment result.
6. Do not output anything other than the JSON structure.
   Given title:
   {given_title}

# Appendix C:Process Description: The Information Entropy of Acquisition Report: a ChatGPT-based Measure

## 1. Risk Text Extraction (From M&A Announcements)

This prompt is used to extract risk text from M&A announcements:

You are tasked with performing the following actions on the given text:

1. Summarize the background.
2. Extract the stock code.
3. Extract the name of the acquiring (leading) company.
4. Extract the name of the target company.
5. Summarize all information in the text related to risks.
6. Extract the announcement number.
   Store the output in JSON format as follows:
   {
   "background": <background>,
   "stkcd": <stock code>,
   "acquirer": <acquiring company name>,
   "target": <target company name>,
   "risk": <all risk-related information>,
   "announcement_no": <announcement number>
   }
   Notes:
7. If relevant information cannot be extracted, directly return null.
8. All risks must be summarized in a list.
9. Answer in Chinese.
10. The response must be directly parsable by json.loads() function.
11. Do not wrap the JSON object inside a markdown JSON code block — output the JSON object directly.
    Background knowledge:
    Stock code: A stock code consists of six digits, generally starting with ‘6’, ‘3’, or ‘0’. In some cases, it may start with ‘SH’ or ‘SZ’, forming an eight-character code. For example, you may see 000001, 000009, 600000, SH606378 in the text.
    Given text: {context_str}

---

## 2. Example of Risk Text Extraction
```json
{
"background": "Announcement of the resolutions of the 2nd Board of Directors' 3rd meeting of Shanghai Jahwa United Co., Ltd., involving the notice of convening the 2002 annual general meeting of shareholders and the decision to change the use of raised funds.",
"stkcd": null,
"acquirer": "Shanghai Jahwa United Co., Ltd.",
"target": "Shanghai Hanxin Industrial Co., Ltd.",
"risk": [
"The acquisition will only take effect after approval by the general meeting of shareholders.",
"The original investment project could not be implemented due to insufficient market acceptance and high upfront costs.",
"The industry faces certain market and operational risks, especially dependency on the upstream supply chain.",
"The investment payback period is 3.85 years, requiring attention to the realization of investment returns.",
"This transaction does not involve personnel placement or arrangements for intangible assets, which may affect subsequent operations.",
"If either party fails to fulfill the contractual agreement, it must bear reasonable costs and expenses."
],
"announcement_no": "Lin 2003-010"
}
```
---

## 3. Python Function for Entropy Calculation

```Python
def cal_entropy(json_file):
    def calculate_entropy_chinese(sentence_list):
        output = {}
        sentence = ''.join(sentence_list)
        
        characters = list(sentence)
        frequency = Counter(characters)
        total_count = len(characters)
        probabilities = [freq / total_count for freq in frequency.values()]
        total_entropy = -sum(p * math.log2(p) for p in probabilities if p > 0)
        
        output['total_entropy'] = total_entropy
 
        individual_entropies = []
        for s in sentence_list:
            characters = list(s)
            frequency = Counter(characters)
            total_count = len(characters)
            probabilities = [freq / total_count for freq in frequency.values()]
            entropy = -sum(p * math.log2(p) for p in probabilities if p > 0)
            individual_entropies.append(entropy)
        
        output['max_entropy'] = max(individual_entropies) if individual_entropies else 0
        output['min_entropy'] = min(individual_entropies) if individual_entropies else 0
 
        return output
    with open(os.path.join(json_file_folder,json_file),'r') as f:
        jsonObj = json.load(f)
    if jsonObj is not None:
        risk = jsonObj.get('risk')
        if risk is not None and len(risk) > 0:
            return calculate_entropy_chinese(jsonObj['risk'])
        else:
            return None
    else:
        return None
```
---

## 4. Result Table: Information Entropy Index of Risk Sentences
Then we get the table below which summarizes the information entropy index of risk related sentences.

| Sentence                                                                                                                             | Information Entropy |
| ------------------------------------------------------------------------------------------------------------------------------------ | ------------------: |
| The acquisition will only take effect after approval by the general meeting of shareholders.                                         |    4.29707932754067 |
| The original investment project could not be implemented due to insufficient market acceptance and high upfront costs.               |    4.73592635062903 |
| The industry faces certain market and operational risks, especially dependency on the upstream supply chain.                         |     4.7606479232901 |
| The investment payback period is 3.85 years, requiring attention to the realization of investment returns.                           |    4.26269239083962 |
| This transaction does not involve personnel placement or arrangements for intangible assets, which may affect subsequent operations. |    4.72004996064481 |
| If either party fails to fulfill the contractual agreement, it must bear reasonable costs and expenses.                              |    4.43660543431788 |

## 5. Risk Level Measurement
Finally, we take the **minimum information entropy** as the risk level measurement.
For the above example, the minimum entropy value is **4.26**, representing the overall risk level.
