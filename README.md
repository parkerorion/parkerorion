{
  "nbformat": 4,
  "nbformat_minor": 0,
  "metadata": {
    "colab": {
      "provenance": []
    },
    "kernelspec": {
      "name": "python3",
      "display_name": "Python 3"
    },
    "language_info": {
      "name": "python"
    }
  },
  "cells": [
    {
      "cell_type": "markdown",
      "source": [
        "Prepare the data, and train and deploy the ML model."
      ],
      "metadata": {
        "id": "-fIKY-lliaXl"
      }
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "id": "KBtBdmDniSgn"
      },
      "outputs": [],
      "source": [
        "# import libraries\n",
        "import boto3, re, sys, math, json, os, sagemaker, urllib.request\n",
        "from sagemaker import get_execution_role\n",
        "import numpy as np\n",
        "import pandas as pd\n",
        "import matplotlib.pyplot as plt\n",
        "from IPython.display import Image\n",
        "from IPython.display import display\n",
        "from time import gmtime, strftime\n",
        "from sagemaker.predictor import csv_serializer\n",
        "\n",
        "# Define IAM role\n",
        "role = get_execution_role()\n",
        "prefix = 'sagemaker/DEMO-xgboost-dm'\n",
        "my_region = boto3.session.Session().region_name # set the region of the instance\n",
        "\n",
        "# this line automatically looks for the XGBoost image URI and builds an XGBoost container.\n",
        "xgboost_container = sagemaker.image_uris.retrieve(\"xgboost\", my_region, \"latest\")\n",
        "\n",
        "print(\"Success - the MySageMakerInstance is in the \" + my_region + \" region. You will use the \" + xgboost_container + \" container for your SageMaker endpoint.\")"
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        " Create the S3 bucket to store your data"
      ],
      "metadata": {
        "id": "DEAL5__ziZbF"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "bucket_name = 'your-s3-bucket-name' # <--- CHANGE THIS VARIABLE TO A UNIQUE NAME FOR YOUR BUCKET\n",
        "s3 = boto3.resource('s3')\n",
        "try:\n",
        "    if  my_region == 'us-east-1':\n",
        "      s3.create_bucket(Bucket=bucket_name)\n",
        "    else: \n",
        "      s3.create_bucket(Bucket=bucket_name, CreateBucketConfiguration={ 'LocationConstraint': my_region })\n",
        "    print('S3 bucket created successfully')\n",
        "except Exception as e:\n",
        "    print('S3 error: ',e)"
      ],
      "metadata": {
        "id": "tzDAN7RLiezO"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "source": [
        "Download the data to your SageMaker instance and load the data into a dataframe"
      ],
      "metadata": {
        "id": "9cF48mSziaBZ"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "try:\n",
        "  urllib.request.urlretrieve (\"https://d1.awsstatic.com/tmt/build-train-deploy-machine-learning-model-sagemaker/bank_clean.27f01fbbdf43271788427f3682996ae29ceca05d.csv\", \"bank_clean.csv\")\n",
        "  print('Success: downloaded bank_clean.csv.')\n",
        "except Exception as e:\n",
        "  print('Data load error: ',e)\n",
        "\n",
        "try:\n",
        "  model_data = pd.read_csv('./bank_clean.csv',index_col=0)\n",
        "  print('Success: Data loaded into dataframe.')\n",
        "except Exception as e:\n",
        "    print('Data load error: ',e)\n"
      ],
      "metadata": {
        "id": "6IQBHcAPjRvT"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "source": [
        "Shuffle and split the data into training data and test data"
      ],
      "metadata": {
        "id": "t9WYX-7DjYk_"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "train_data, test_data = np.split(model_data.sample(frac=1, random_state=1729), [int(0.7 * len(model_data))])\n",
        "print(train_data.shape, test_data.shape)"
      ],
      "metadata": {
        "id": "ea91l71_jYIx"
      },
      "execution_count": null,
      "outputs": []
    }
  ]
}
