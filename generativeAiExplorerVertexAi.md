# Generative AI Explorer - Vertex AI

# getting started
!pip install google-cloud-aiplatform==1.36.2 --upgrade --user
### Automatically restart kernel after installs so that your environment can access the new packages
import IPython

The model sizes are as follows:
Bison: The best value in terms of capability and cost.
Gecko: The smallest and cheapest model for simple tasks.

app = IPython.Application.instance()
app.kernel.do_shutdown(True)
# prompt design
!pip install google-cloud-aiplatform==1.36.2 --upgrade --user
!pip install google-cloud-aiplatform protobuf==3.19.3 --upgrade --user
- Be concise
prompt = "SOMETHING NOT CONCISE"
print(generation_model.predict(prompt=prompt, max_output_tokens=256).text)
- Be specific, and well-defined
- Ask one task at a time
- Improve response quality by including examples
- Turn generative tasks to classification tasks to improve safety
# get started with Vertiex AI Studio
- note
`Temperature controls the degree of randomness in token selection. Lower temperatures are good for prompts that expect a true or correct response, while higher temperatures can lead to more diverse or unexpected results. With a temperature of 0 the highest probability token is always selected`
- task 1
Artificial Intelligence > Vertex AI> Vertex AI Studio> Overview
- task 2
gcloud storage cp gs://spls/gsp154/video/train.mp4 gs://<Your-Cloud-Storage-Bucket>
- task 3
Vertex AI Studio > Overview page. Click Open for Language Powered by Gemini
- task 4


cloud-ai-platform-c2813483-079e-4299-a854-f1d0938590a6

gcloud storage cp gs://spls/gsp154/video/train.mp4 gs://cloud-ai-platform-c2813483-079e-4299-a854-f1d0938590a6