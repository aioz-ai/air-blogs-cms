---
last_modified_on: "2023-07-14"
title: Variational Autoencoders (VAE)
description: Variational Autoencoder
series_position: 39
author_github: https://github.com/aioz-ai
tags: ["type: insight", "level: medium"]
---

Variational Autoencoders (VAEs) are a type of generative model that combine the encoding and decoding capabilities of autoencoders with probabilistic inference. They learn a compressed representation of input data while also modeling the underlying distribution of the data, enabling them to generate new samples.

## Why were VAEs created?
While traditional autoencoders are effective at feature extraction and reconstruction, they often neglect the distribution of the latent features. This can lead to overfitting and the generation of unrealistic samples. VAEs were introduced to address this limitation by incorporating probabilistic concepts to regulate the distribution of the latent space.

Similarities and Differences between Autoencoders and VAEs
VAEs share similarities with autoencoders in terms of their encoding and decoding processes. However, VAEs utilize probability theory to create a more complex and capable representation space for data generation.

## Variational Autoencoder Architecture

In contrast to autoencoders, which learn the compressed representation directly, VAEs learn the probability distributions of the latent variables. This enables deeper representation learning and provides control over the generative process.

Here's an example implementation of a VAE in Python using PyTorch:

```
import torch
import torch.nn as nn
import torch.optim as optim
from torch.distributions import Normal

# Define the VAE model
class VAE(nn.Module):
    def __init__(self, input_dim, latent_dim):
        super(VAE, self).__init__()
        
        # Encoder layers
        self.encoder = nn.Sequential(
            nn.Linear(input_dim, 128),
            nn.ReLU(),
            nn.Linear(128, 64),
            nn.ReLU(),
        )
        
        # Latent space layers
        self.fc_mu = nn.Linear(64, latent_dim)
        self.fc_logvar = nn.Linear(64, latent_dim)
        
        # Decoder layers
        self.decoder = nn.Sequential(
            nn.Linear(latent_dim, 64),
            nn.ReLU(),
            nn.Linear(64, 128),
            nn.ReLU(),
            nn.Linear(128, input_dim),
            nn.Sigmoid(),
        )
    
    def encode(self, x):
        hidden = self.encoder(x)
        mu = self.fc_mu(hidden)
        logvar = self.fc_logvar(hidden)
        return mu, logvar
    
    def decode(self, z):
        output = self.decoder(z)
        return output
    
    def forward(self, x):
        mu, logvar = self.encode(x)
        z = self.reparameterize(mu, logvar)
        output = self.decode(z)
        return output, mu, logvar
```
## Loss Function in VAEs
The loss function of a VAE consists of two components: the reconstruction loss and the regularization loss.

The reconstruction loss measures the discrepancy between the input and the reconstructed output. Common loss functions used for reconstruction include mean squared error or mean absolute error. For binary data, binary cross-entropy can be used.

The regularization loss, calculated using the Kullback-Leibler (KL) divergence, encourages the latent variables to follow a standard normal distribution. It helps control the distribution of the latent space and prevent overfitting.

Here's an example implementation of the loss function (ELBO) in Python using PyTorch:

```
# Define the loss function (ELBO)
def loss_function(output, target, mu, logvar):
    # Reconstruction loss (binary cross-entropy)
    recon_loss = nn.functional.binary_cross_entropy(output, target, reduction='sum')
    
    # KL divergence loss
    kl_loss = -0.5 * torch.sum(1 + logvar - mu.pow(2) - logvar.exp())
    
    # Total loss
    total_loss = recon_loss + kl_loss
    return total_loss
```

## Reparameterization Trick
To enable backpropagation through the sampling step of the latent variables, VAEs use a reparameterization trick. Instead of directly sampling from the probability distribution, a sample is generated from a standard normal distribution and transformed based on the learned mean and variance from the encoder.

Here's an example implementation of the reparameterization trick in the VAE model:

```
def reparameterize(self, mu, logvar):
    std = torch.exp(0.5 * logvar)
    eps = torch.randn_like(std)
    z = mu + eps * std
    return z
```

In summary, Variational Autoencoders (VAEs) extend the capabilities of autoencoders by incorporating probabilistic inference. They learn a compressed representation of data while also modeling the distribution of the latent space. VAEs have various applications in generating new samples and learning meaningful representations.