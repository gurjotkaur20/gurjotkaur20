# 1. AI Fundamentals

## 1.1 Introduction to AI

AI has the transformative potential to infuse every industry due to its capacity to enhance:
- **Efficiency** - Streamline processes and automate tasks
- **Decision-making** - Provide data-driven insights
- **Innovation** - Uncover valuable insights from vast datasets

By leveraging advanced algorithms, machine learning, and data analytics, organizations can:
- Streamline processes
- Automate repetitive tasks
- Uncover valuable insights from vast data sets

### Industry Applications

### 1.1.1 Call Centers
- AI virtual agents
- Intent extraction
- Sentiment analysis

### 1.1.2 Retail
- Store analytics and traffic trends
- Customer shopping patterns
- Real-time aisle occupancy monitoring
- Basket analysis

### 1.1.3 Manufacturing
- Product design optimization
- Production optimization
- Quality control improvement
- Waste reduction
- Enhanced safety protocols

---

## 1.2 AI Transformation Across Industries

### 1.2.1 Healthcare Industry

AI is revolutionizing drug discovery and medical devices by:

1. **Biological Modeling**
   - Modeling biological systems
   - Studying diseases
   - Discovering drug targets

2. **Lab Automation**
   - Using sensors and robotics for high-throughput screening
   - Automating drug formation processes

3. **In-Silico Drug Discovery**
   - AI and computing for molecular generation
   - Structure prediction
   - Virtual screening
   - Predictive molecular dynamics

#### 1.2.1.1 Use Case: Invenio Imaging

Invenio Imaging is a medical device company developing technology that enables surgeons to evaluate tissue biopsies in the operating room immediately after samples are collected.

**Key Features:**
- Uses a cluster of NVIDIA RTX GPUs to train AI neural networks
- Real-time image analysis layered on tissue images
- Physicians can quickly determine cancer cell types in biopsy images
- Analyze different molecular subtypes of cancer within tissue samples
- Predict patient response to chemotherapy
- Determine if tumors have been successfully removed during surgery

---

### 1.2.2 Finance Industry

AI is transforming the financial sector by:

1. **Robo-Advisors**
   - Algorithm-powered investment portfolio selection and management

2. **Video Analytics**
   - Intelligent threat detection
   - ATM security monitoring

3. **Fraud Detection**
   - Deep learning-based anomaly detection
   - Real-time notifications
   - Enhanced accuracy for financial transactions

#### 1.2.2.1 Use Case 1: Deutsche Bank

Frankfurt-based Deutsche Bank is working with NVIDIA to accelerate AI and machine learning on transformational applications.

**Key Features:**  
With NVIDIA's Omniverse platform ​and open computing platform ​for building and operating metaverse applications, this bank is exploring how to: 

- Enhance employee and customer engagement
- Interactive 3D virtual avatars
- HR-related question responses

Deutsche Bank and NVIDIA are testing ​a collection of large language models ​targeted at financial data called ​Financial Transformers or Finformers. These will detetct 
- early warning signs for counterparty financial transaction risks
- Faster data retrieval
- Identification of data quality issues

#### 1.2.2.2 Use Case 2: American Express

American Express leverages deep learning through the NVIDIA GPU computing platform to combat fraudulent activity.

**Key Features:**  
Using advanced AI models 
- To detect anomalous patterns and transactions
- Enhanced real-time fraud detection system
- Improved accuracy in fraud detection
- Better protection for customers and merchants

---

### 1.2.3 Automotive Industry

AI is being used in the automotive industry from design to autonomous vehicles by:

1. **Design & Simulation**
   - Design visualization
   - Engineering simulation

2. **Digital Twins**
   - Industrial digital twins
   - Virtual showrooms

3. **Autonomous Systems**
   - Intelligent assistants
   - Autonomous driving systems
   - Autonomous parking systems

---

## 1.3 Evolution of AI

### 1.3.1 The AI Hierarchy

```
Artificial Intelligence (AI)
│
├── Machine Learning (ML)
│   │
│   ├── Deep Learning (DL)
│   │   │
│   │   ├── Generative AI (GenAI)
│   │   │   │
│   │   │   ├── Agentic AI
│   │   │   │
│   │   │   └── Physical AI
│   │
│   ├── Supervised Learning
│   ├── Unsupervised Learning
│   └── Reinforcement Learning
│
└── Rule-Based AI / Expert Systems
```

### 1.3.2 Timeline and Evolution

#### 1.3.2.1 AI (1950s onwards)
- Broad field of study focused on using computers to do things that require human-level intelligence
- Using machines ​to mimic human abilities to learn, analyze, and predict. 
- Early applications: Games like Tic-Tac-Toe and Checkers
- Inspired sci-fi movies with both hope and fear

#### 1.3.2.2 Machine Learning (1980s)
- An approach to AI using statistical techniques
- Constructs models from observed data
- Uses large datasets and sophisticated statistical methods to train a model to predict ​outcomes from new incoming information. 
- **Example**: Email spam filters
- Revolution enabled by: Smartphones, webcams, social media, and sensors
- Challenge: Understanding and extracting insights from massive datasets (Big Data)

#### 1.3.2.3 Deep Learning (Around 2010)
Deep learning ​uses artificial neural networks to learn from vast amounts of data to solve AI problems

- Real breakthroughs due to:
  - Advances in hardware capabilities
  - Availability of large datasets
  - Improvements in training algorithms
  - Automated creation of feature extractors
  - Large complex deep neural networks (DNNs)

**Perception AI (2012):**
- AlexNet won the ImageNet Large-Scale Visual Recognition Challenge
- One of the first deep convolutional neural networks to leverage GPUs for training
- Dramatically reduced training time and made it feasible to train on large datasets
- Marked the beginning of Perception AI era

#### 1.3.2.4 Generative AI (2020s)
A type of AI that uses machine learning algorithms to learn patterns and ​trends from the training data using neural networks to create new content that mimics ​human-generated content

- Emerged within one decade of Deep Learning
- Era of Large Language Models (LLMs)
- Systems with surprisingly human-like intelligence and capabilities

**Key Milestone - ChatGPT (2022):**
- Launched by OpenAI, five years after the Transformer architecture introduction in 2017
- Models like ChatGPT able to create new content from human input

**Applications:**
- Chatbots
- Virtual assistants
- Content generation
- Translation services
- Other AI applications impacting industries and daily life

#### 1.3.2.5 Agentic AI
A form of artificial intelligence focused on creating **autonomous systems** (AI agents) that can:
- Reason independently
- Plan complex actions
- Act autonomously to achieve complex tasks
- Operate with minimal or no human intervention
- **Collaborate** with other AI agents
- Form **multi-agent systems** to tackle cross-domain problems

#### 1.3.2.6 Physical AI
Another form of AI that enables machines (robots) to:
- Perceive the physical environment
- Understand and interact with real-world
- Integrate AI algorithms with sensors and actuators
- Make decisions and execute physical tasks autonomously

**Types of Physical AI Robots:**

1. **Infrastructure Robots**
   - Used in factories and warehouses
   - Automate material handling and manufacturing processes

2. **General Purpose Robots**
   - Include humanoid robots
   - Capable of performing diverse tasks across different domains

3. **Transportation Robots**
   - Autonomous cars
   - Robo-taxis
   - Self-driving vehicles for mobility and logistics

**Training and Deployment:**
- Trained using high-fidelity simulations and digital twins
- Allows systems to safely learn and practice new skills at superhuman speed
- Deployed in the real world with proven competency

---

#### 1.3.2.7 AI Factories

AI factories are designed to handle the increasing demands for compute resources as AI models scale up in complexity.

**Understanding AI Scaling Laws:**
AI scaling laws describe how model performance improves as we increase three key factors:
- Size of the models
- Amount of training data
- Compute power available

**The Three Scaling Laws:**

1. **Pre-training Scaling**
   - Larger models trained on more data with more compute yield better results
   - Drove the initial wave of investment in massive AI training infrastructure

2. **Post-training Scaling**
   - Highlights the need for additional compute to fine-tune and adapt models for specific real-world applications
   - Further increases infrastructure demands

3. **Test Time Scaling (Long Thinking)**
   - Driven by reasoning models that use much more compute during inference
   - Generate and evaluate multiple intermediate steps before producing a final answer
   - Consume up to 100 times more compute than traditional inference

**Why Industries Need AI Factories:**
- These scaling laws drive exponential increase in demand for specialized, efficient compute infrastructure
- Far beyond what traditional data centers were designed to handle
- Purpose-built AI factories are needed to keep up with the pace of innovation
- Enable delivery of next-generation intelligent systems at scale

---

## 1.4 AI Workflow

An **AI workflow** (also called machine learning or data science workflow) is a sequence of tasks and processes that data scientists, ML engineers, and AI practitioners follow to develop, train, deploy, and maintain AI models.

### 1.4.1 Four Main Steps

#### 1.4.1.1 Data Preparation

Collecting, cleaning, and pre-processing raw data to make it suitable for training and evaluating AI models

#### 1.4.1.2 Model Training

Using a machine learning or deep learning model to learn patterns and relationships within a labeled dataset

**Key Points:**
- Most compute-intensive phase of the AI workflow
- Usually benefits most from increased hardware resources
- Other phases (data prep, inference) require less computation

**Optimization Techniques:**
- Mixed-precision data formats (FP8, FP16) for efficiency
- Maintains model accuracy while improving speed
- Batch processing and parallel training

#### 1.4.1.3 Model Optimization

Fine-tuning and enhancing the performance of the AI model to make it more accurate, efficient, and suitable for its intended use case

**Goals:**
- Improve model accuracy
- Increase inference speed
- Reduce model size
- Enhance efficiency for deployment

#### 1.4.1.4 Inference

Using a trained ML or deep learning model to make predictions, decisions, or generate outputs based on new, unseen data

**Applications:**
- Real-time predictions
- Batch processing
- Online/streaming predictions
- Embedded systems

**Considerations:**
- Latency requirements
- Throughput needs
- Resource constraints
- Accuracy requirements

---

### 1.4.2 AI Workflow Steps and Models

| Workflow Step | Tools/Models |
|---------------|--------------|
| **Data Preparation** | RAPIDS, Pandas, Apache Spark |
| **Model Training** | TensorFlow, PyTorch, Scikit-learn, TAO |
| **Model Optimization** | TensorRT, ONNX Runtime |
| **Inference** | NVIDIA Triton inference server  |

---

## 1.5 Generative AI

Generative AI refers to a subset of artificial intelligence that focuses on creating data or content, such as images, text, and multimedia, based on patterns and examples from a given dataset.

### 1.5.1 Foundational Models

Foundational models serve as the basis or foundation for the creation and evolution of generative AI systems, providing the initial framework for understanding complex language structures, semantics, and contextual nuances.

**Key Characteristics:**
- Consist of AI neural networks trained on massive unlabeled datasets, generally with unsupervised learning.
- Rely on finding patterns and structures in the data independently without requiring labeled data to train a model

### Examples

| Model | Description |
|-------|-------------|
| **DALL-E** | Creates realistic images from text descriptions. Can be used for image synthesis tasks such as image captioning, image editing, and image manipulation. |
| **eDiff-I** | A stable diffusion model for synthesizing images given text which generates photorealistic images corresponding to any input text prompt. |
| **Llama 2** | Can be used for generating diverse and high-quality natural language text, making it valuable for various tasks such as content creation, language understanding, and conversational AI applications. |
| **NVIDIA GPT** | A family of production-ready large-language models (LLMs) that can be tuned to build enterprise generative AI applications. Can perform a range of tasks from creating product descriptions, answering customer queries, and writing code. |
| **GPT-4** | Gives applications the ability to create human-like text and content, images, music, and more, and answer questions in a conversational manner. |

---

### 1.5.2 Large Language Models (LLM)

Foundation models provide the basic foundation for LLM.

**Key Technology:**
- Language models utilize a specialized neural network known as the transformer to grasp patterns and relationships within textual data
- Undergo pre-training on extensive text datasets and can be fine-tuned for specific tasks

---

### 1.5.3 Challenges and Considerations in Generative AI

While generative AI offers significant benefits, its adoption does not come without challenges.

#### 1.5.3.1 Benefits
- Increased efficiency
- Cost savings
- Enhanced capabilities and automation

#### 1.5.3.2 Implementation Challenges and Risks

1. **Data Privacy & Security**
Generative AI use cases in the healthcare and financial sectors should be monitored very closely to forestall any money-related or sensitive data leakages.

2. **Intellectual Property Rights & Copyright**
Generative AI platforms should mitigate copyright infringement of the creator's work.

3. **Biases, Errors, and Limitations**
Generative AI is just as prone to biases as humans are because in many ways it is trained on our own biases.

4. **Ethical Implications**
Determining responsibility for the outputs of generative AI can be challenging. If AI systems generate harmful content, it may be unclear who bears responsibility – the developers, the users, or the technology itself.

5. **Malevolent Activities**
There is no state-of-the-art know-how that wrongdoers can't put to their evil uses, and generative AI is not an exception where fraudulent scams of various kinds can be created.

**Responsibility and Guardrails:**
- It's our responsibility to build using guardrails, or rules, to mitigate inappropriate outcomes
- Establishing safeguards ensures responsible and safe deployment of generative AI systems
- Proper governance frameworks are essential for sustainable implementation

---

### 1.5.4 Building Custom Generative AI Models

**Building Foundation Models from Scratch:**
- Requires a lot of data and compute resources
- Very costly process
- Typically undertaken by large technology companies, research institutions, and well-funded startups

**Customizing Pre-trained Models:**
- Requires less data, resources, and expertise
- Involves feeding the model with domain-specific data and tasks
- Adjusts model parameters accordingly
- Less resource-intensive compared to building from scratch
- Requires knowledge of model capabilities, data format, and evaluation metrics

**Industry Approach:**
- Many organizations choose the cost-effective approach of leveraging existing foundational models
- Models offered by AI research companies can be fine-tuned to specific needs
- Fine-tuning approach balances resource constraints with customization requirements

---

### 1.5.5 Building Generative AI Applications

**Workflow Overview:**

```
┌─────────────────────────────────────────┐
│       DATA PREPARATION                  │
│  ┌──────────────┐  ┌──────────────┐    │
│  │   Data       │  │   Data       │    │
│  │ Acquisition  │─▶│ Curation     │    │
│  └──────────────┘  └──────────────┘    │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│  TRAINING AND CUSTOMIZATION             │
│  ┌──────────────┐  ┌──────────────┐    │
│  │ Pre-training │  │Customization │    │
│  │              │─▶│              │    │
│  └──────────────┘  └──────────────┘    │
│         │                    │          │
│         └─────────┬──────────┘          │
│                   ▼                     │
│         ┌──────────────────┐            │
│         │ Model Evaluation │            │
│         └──────────────────┘            │
└────────────────┬────────────────────────┘
                 │
                 ▼
┌─────────────────────────────────────────┐
│       DEPLOYMENT                        │
│  ┌──────────────┐  ┌──────────────┐    │
│  │  Inference   │  │ Guardrails   │    │
│  │              │─▶│              │    │
│  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────┘
```

**Steps Involved:**

#### Data Preparation

**Data Acquisition:**
- Collecting and preparing data for training and fine-tuning the LLM
- Data sources include public datasets, web scraping, user-generated content, and proprietary data
- Data must be diverse and representative of the target domain

**Data Curation:**
- Cleaning, filtering, and organizing data for training and fine-tuning
- Ensures data quality and relevance for the specific use case

#### Training and Customization

**Pre-training:**
- Exposing the model to a vast corpus of text data
- Facilitates learning of language patterns, relationships, and representations
- Typically incorporates a foundational model as the starting point

**Customization:**
- Adapting a generic model to specific requirements of a task or domain
- Improves accuracy, efficiency, and effectiveness for targeted applications

**Model Evaluation:**
- Assessing the performance and effectiveness of the machine learning model
- Measures how well the model has learned from training data
- Evaluates accuracy on unseen or new data

#### Deployment

**Inference:**
- Deploying the trained model for inference
- Model processes input data and produces output (classifications, predictions, recommendations)
- Output depends on the specific task the model was trained for

**Adding Guardrails:**
- Crucial for fostering responsible AI practices
- Mitigates risks associated with misuse or misinterpretation of generated text
- Ensures ethical, safe, and responsible use of the model

---

**Last Updated**: June 2026
