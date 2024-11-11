

- code to delete files added on a specific date in a directory
    
    ```python
    import os
    
    folder_path = os.path.join(root_dir, f'data/Processed Data/images/val')
    # Get a list of files in the folder
    files = os.listdir(folder_path)
    
    # Iterate over each file
    for file in files:
        file_path = os.path.join(folder_path, file)
        
        # Get the file metadata
        file_stats = os.stat(file_path)
        
        # Access the metadata properties
        file_size = file_stats.st_size  # File size in bytes
        file_mtime = file_stats.st_mtime  # Last modification time
        
        # If file has been modified on given date, delete it
        check_date = datetime.date(2023, 6, 20)
        if datetime.date.fromtimestamp(file_mtime)==check_date:
            print(file_path.split('/')[-1])
            os.remove(file_path)
    ```
    
- return numpy arrays(imgs) from web endpoint using fastapi
    
    ```python
    #ref: https://www.appsloveworld.com/numpy/100/59/how-to-return-a-numpy-array-as-an-image-using-fastapi
    
    import io
    import imageio
    from imageio import v3 as iio
    from fastapi import Response
    
    @app.get("/image", response_class=Response)
    def get_image():
        im = imageio.imread("test.jpeg") # 'im' could be an in-memory image (numpy array) instead
        with io.BytesIO() as buf:
            iio.imwrite(buf, im, plugin="pillow", format="JPEG")
            im_bytes = buf.getvalue()
            
        headers = {'Content-Disposition': 'inline; filename="test.jpeg"'}
        return Response(im_bytes, headers=headers, media_type='image/jpeg')
    
    ```
    
- install via shell commands in modal image
    
    ```
    # .run_commands(
                    #  pip install git+https://$AUTH_USER:$AUTH_PASSWORD@github.com/FarMart-Tech/ds-common-utils@lavish-dev
                #     f"git clone -b dev https://AUTH_USER:AUTH_PASSWORD@github.com/FarMart-Tech/grain-instance-segmentation.git",
                #     "cd grain-instance-segmentation && pip install -e .",
                #     secrets=[modal.Secret.from_name("deepam-github-secret")],
                # )
    ```
    
    
- IDE documentation
    
    # Problem Understanding
    
    The primary objective of this project is effectively classify and verify business documents uploaded in the system in the entire trail of sauda fulfillment. The system should be able to extract structured information from the business documents.
    
    This can help automate the information extraction process and ensure KYC/Onboarding and Transaction documents are compliant. Additionally, it will facilitate the automation of information in the document for payment execution and smooth onboarding processes.
    
    *(Although there exist 3rd party services that can do this for us, using them poses data-related risks and overpriced costs.)*
    
    The PRD for the service we have deployed for the Tech team is [here](https://farmart.atlassian.net/wiki/spaces/MLF/pages/333447169/Document+Verification+and+Input+Optimisation+-+For+Compliance+and+Productivity+Enhanceent). It explains exactly what data points we aim to extract and from which document.
    
    This will be deployed as an API service internally and given a document should -
    
    - classify documents.
    - check its validity.
    - extract data points from them.
    
    # Abstract
    
    While classifciation of documents is easier, information extraction is a more challenging task since it requires complex functions such as reading text and a holistic understanding of the document.
    
    For information extraction, we use the **donut**, a transformer based model. Information extraction is done via visual question answering (VQA) since it can be generalised for new documents, and provide new information we may need from the same document without having to re-train the entire model.
    
    Currently we are deploying a service that automates form-filling for kyc documents (aadhaar, pan, and gst documents). Next iterations will involve weight slips, and other documents as per PRD.
    
    The **risks** associated with our approach are as follows-
    
    - Donut is a generative encoder-decoder model. It can reliably images with **1. clear** and **2. correctly oriented** text, especially when information is **3. present as key-value pairs**. But when any of these 3 is not satisfied permormance is low.
    - Since its generative we also have risks like *lack of control* over the outputs & possibly *nonsensical outputs*.
    
    We assume that list of documents will remain same for onboarding and payment process, hence we can overfit the model to learn layout information about specific data-points across categories and minimize nonsensical data output.
    
    Although we could trained the same model for classification task too, we opt for a more light-weight soultion. We extract features from images using the [CLIP](https://arxiv.org/pdf/2103.00020.pdf) model and then classify those features using a classical machine learning model like SVM/Logistic regressor.
    
    # Literature Review
    
    Some of the important public datasets for document parsing and VQA are → SROIRE, FUNSD and DocVQA datasets.
    
    There are 3 feature categories that can be extracted for document intelligence tasks - *text*, *layout* (bounding box) and *image.* The text and layout features are typically extracted using an off the shelf OCR and image features using CNN/transformers.
    
    Initally only text features were extracted using OCR ([EasyOCR](https://github.com/JaidedAI/EasyOCR), [PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR)) and a language model like BERT was placed on top to extract structured information from those features.
    
    Then inspired by NLP, we saw pretraining+finetuning paradigm come to the fore. Inspired by BERT, [LayoutLM](https://arxiv.org/pdf/1912.13318.pdf) introduced pre-training in document intelligence (Masked visual language modelling (MVLM) and Multi-label Document Classification) and finetuning for different tasks. It extracts text and layout features using OCR for pretraining and adds visual features using Faster R-CNN during finetuing. [StructLM](https://arxiv.org/pdf/2105.11210.pdf) improves on LayoutLM model but is built along the same principles. [LayoutLMv2](https://arxiv.org/pdf/2012.14740.pdf) uses OCR and a visual encoder to get features in all 3 modalitites which are used to pretrain a transformer architecture in MVLM, text-image matching and text-image alignment pretraining tasks. [LayoutLMv3](https://arxiv.org/pdf/2204.08387.pdf) has unified text and image masking objectives. It is pre-trained with discrete token reconstructive objectives of Masked Language Modeling (MLM), Masked Image Modeling (MIM) and Word-Patch Alignment (WPA) objective to learn cross-modal alignment by predicting whether the corresponding image patch of a text word is masked.
    
    (Add TILT, Formnet, BROS, DocFormer)
    
    Very recently, models like [donut](https://arxiv.org/pdf/2111.15664.pdf) an [pix2struct](https://arxiv.org/pdf/2210.03347.pdf) use only image features provide a single end-to-end solution and remove the OCR dependency. Donut is trained to read all texts in the image in reading order (from top-left to bottom-right, basically). The objective is to minimize cross-entropy loss of next token prediction by jointly conditioning on the image and previous contexts. This task can be interpreted as a pseudo-OCR task.
    
    Hence we have trained and deployed the *donut* architecture, because it has several advantages over other methods:
    
    - no object detection required : these models have been pretrianed to read text from documents having varying backgrounds and hence are robust.
    - no ocr dependency : dependency on OCR is bad because of high computational costs and error propagation. (see Fig 6)
    - simplicity and flexibility : it is a end-to-end model pretrained to read text from documents and can be finetuned in 3 tasks (classification, information extraction, visual question answering).
    
    Figure 1. Pipeline of Donut
    
    # Data
    
    ### Sources
    
    The data we used has been retrieved from the tech database. The following tables were used -
    
    - kyc documents : merchant_verification_docs
    - weight slips : order_delivery_documents
    
    Images of documents from these were uploaded to label studio and based on type of document, labelling is done manually. Then this annotated data is exported to csv files which are then converted to the format comaptible for donut.
    
    ### Structure
    
    Our data structure follows this structure: Images are splitwise present in images folder, and the question answer informtion ins contained in the metadata files (jsonl files).
    
    ```
    data
    ├── images
    │   ├── test/
    │   └── train/
    │
    ├── train.jsonl
    └── test.jsonl
    ```
    
    Donut uses jsonlines format for training. The data files contain json objects in each line. One line in a train.jsonl file would look like this -
    
    ```
    {
      "file_name": "1645254005551-test.jpg",
      "ground_truth": "{\"gt_parses\":
            [{\"question\": \"What is this document?\", \"answer\": \"bank details\"}]
          }"
    }
    ```
    
    ### Data Exploration
    
    For version-1 we have lablled over 500 images generating around 3500 question answer pairs. The notebook used to prepare data from the label studio exports is [here](https://github.com/FarMart-Tech/IDE/blob/deepam_dev/notebooks/01_prepare_data_from_label_studio.ipynb).
    
    We have currently labelled documents in these categories: (refer [PRD](https://farmart.atlassian.net/wiki/spaces/MLF/pages/333447169/Document+Verification+and+Input+Optimisation+-+For+Compliance+and+Productivity+Enhanceent) for in-depth information)
    
    Figure 3: Document type distribution
    
    To prevent data-leakage into the test sets, we have split our data into train-test-val at the image level, so that an image is present in only one split.
    
    Figure 4: train-test-val split of data
    
    # Modelling
    
    ### Donut
    
    The architecture of Donut is quite simple, which consists of a Transformer based visual encoder and textual decoder modules. Note that Donut does not rely on any modules related to OCR functionality but uses a visual encoder for extracting features from a given document image. The following textual decoder maps the derived features into a sequence of subword tokens to construct a desired structured format (e.g., JSON). Each model component is Transformer-based, and thus the model is trained easily in an end-to-end manner. The overall process of Donut is illustrated in Figure 5. The authors of the paper have used a Swin-Transformer as the visual encoder and BART as the language decoder.
    
    Figure 5. The pipeline of Donut. The encoder maps a given document image into embeddings. With the encoded embeddings, the decoder generates a sequence of tokens that can be converted into a target type of information in a structured form
    
    The visual encoder converts the input document image  into a set of embeddings , where  is feature map size or the number of image patches and  is the dimension of the latent vectors of the encoder. Given the , the textual decoder generates a token sequence , where  is an one-hot vector for the i-th token,  is the size of token vocabulary, and  is a hyperparameter, respectively.
    
    The model is trained to read all texts in the image in reading order (from top-left to bottom-right, basically). The objective is to minimize cross-entropy loss of next token prediction by jointly conditioning on the image and previous contexts. The dataset used for this was the IIT-CDIP, which is a set of 11M scanned english document images, along with synthetic data they generated (~2M).
    
    The pretrained model can then be finetuned for different tasks (doc classification, doc parsing, VQA). Since we want to finetune on the VQA task, we pick a [open-source model](https://huggingface.co/naver-clova-ix/donut-base-finetuned-docvqa) that has been finetuned on the DocVQA dataset as our baseline. This dataset consists of 50K questions defined on more than 12K documents.
    
    We then finetune it on our custom dataset. Go checkout the [training script](https://github.com/FarMart-Tech/IDE/blob/deepam_dev/IDE/train.py) and the [testing script](https://github.com/FarMart-Tech/IDE/blob/deepam_dev/IDE/test.py) for more details on finetuning.
    
    Figure 6. Comparison of BERT, LayoutLMv2 and Donut on CORD. The performances (i.e., speed and accuracy) of the OCR-based models extrsemely varies depending on what OCR engine is used (left and center). Donut shows robust performances even in a low resourced situation showing the higher score only with 80 samples (right)
    
    # Deployment
