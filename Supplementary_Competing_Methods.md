# Supplementary Competing Methods

This document provides detailed implementation settings for the competing methods evaluated in the main article. These details were moved from the main text to the supplementary material during proof correction to comply with the IEEE JBHI page-limit requirement.

\subsubsection{Competing Methods}


To evaluate the effectiveness of our proposed UJA framework, we compared it with several representative methods, including traditional machine learning models, domain generalization networks, and domain adaptation approaches. These methods differ in feature extraction strategies and domain alignment mechanisms, providing a comprehensive reference for assessing the advantages of our unified attention-based and adversarial alignment scheme.


\textbf{Traditional machine learning methods} include:

\textbf{BC-SVM-N}: This method computes seven node-level brain connectivity features based on the FC matrix constructed using Pearson correlation. These features include node degree, strength, clustering coefficient, local efficiency, betweenness centrality, eigenvector centrality, and modularity, as defined in \cite{rubinov2010complex}. The features for each ROI are concatenated into a high-dimensional feature vector and used to train a linear Support Vector Machine (SVM) classifier. Feature normalization (z-score) and a grid search are used to optimize the penalty parameter~$C$.

\textbf{BC-SVM-G}: This method derives five global topological features from the FC matrix, including graph density, global efficiency, assortativity coefficient, characteristic path length, and transitivity. These features are aggregated into a five-dimensional vector per subject and input into a linear SVM for classification. As with BC-SVM-N, standardization and grid-based hyperparameter optimization are applied.

\textbf{BC-SVM-N-G}: This approach combines the node-based and graph-based features into a single concatenated vector, capturing both local and global connectivity characteristics. The resulting hybrid representation is used as input to a linear SVM, allowing the model to leverage complementary information from both feature spaces.


XGBoost~\cite{shi2021multivariate}: For each subject, a PCC-based functional connectivity matrix is constructed, and the upper triangular elements are vectorized to form a 6,670-dimensional feature vector. This vector is used to train an XGBoost classifier. Following previous studies, we set the maximum tree depth to 5, the minimum child weight to 1, and use a subsampling ratio of 0.8 for both instances and features. The learning rate is set to 0.1, and an L1 regularization term with a coefficient of 0.01 is applied to reduce overfitting.



\textbf{MaLRR}~\cite{wang2019identifying}: This method addresses multi-site heterogeneity by projecting source and target data into a shared low-rank subspace using low-rank representation (LRR) decomposition. The aligned features are subsequently classified using a linear SVM. This approach aims to mitigate domain shift by learning a shared low-rank subspace that captures common latent structures across sites.


\textbf{Deep learning-based methods} include:

\textbf{Wck-CNN}~\cite{jie2020designing}: This model introduces weighted correlation kernels within a convolutional neural network framework to capture hierarchical interactions among brain regions. The architecture consists of four convolutional layers followed by two fully connected layers with 64 and 32 neurons, respectively. A dropout rate of 0.5 is applied after each fully connected layer, and ReLU is used as the activation function throughout.

\textbf{LSTM}~\cite{li2018brain}: The model consists of two stacked LSTM layers with 256 hidden units each, followed by a fully connected softmax output layer. It is trained using the Adam optimizer with an initial learning rate of 0.001, decayed by a factor of 0.1 every 50,000 steps, for a total of 200,000 steps. The batch size is set to 32. During training, input sequences are cropped into 40×90 clips with 20-timepoint overlap for data augmentation.

\textbf{STNet} \cite{wang2019spatial}: STNet is a convolutional-recurrent neural network designed for rs-fMRI analysis. It segments BOLD signals into 54 overlapping windows (length = 30, step = 2), with each window processed by a three-layer convolutional module (16, 8, and 1 channels) to extract spatial features and detect network hubs. Temporal dynamics are modeled using three stacked LSTM layers (16, 8, and 4 units). A fully connected softmax layer performs final classification. The model is trained with the Adam optimizer (batch size = 16, 200 epochs) and includes batch normalization and dropout (rate = 0.5) throughout.

\textbf{ComBat} \cite{johnson2007adjusting,fortin2018harmonization}: ComBat is a widely used statistical harmonization method that corrects batch or site effects using an empirical Bayes framework. In this study, ComBat was applied to the functional connectivity features to remove site-related variability across imaging centers. The harmonized features were subsequently used for classification using two classifiers: the Transformer-based model proposed by Dai et al\cite{dai2024classification}, and a Support Vector Machine (SVM). This baseline evaluates the effectiveness of traditional statistical harmonization compared with domain adaptation approaches.

\textbf{DANN} \cite{ganin2015unsupervised}: A domain-adversarial neural network composed of a shared feature generator, a label predictor, and a domain classifier connected via a gradient reversal layer to encourage domain-invariant features. The domain classifier consists of three fully connected layers (1024~$\rightarrow$~1024~$\rightarrow$~2). Training uses stochastic gradient descent with momentum 0.9, batch size 128, and an initial learning rate of 0.01, decayed as $\mu_p = 0.01 / (1 + 10p)^{0.75}$. The domain adaptation factor $\lambda$ increases during training via $\lambda_p = \frac{2}{1 + e^{-10p}} - 1$.


\textbf{UGDAN} \cite{wang2020unsupervised}: UGDAN is an unsupervised graph domain adaptation model designed for neurodevelopmental disorder diagnosis across sites and disorders. It consists of three core modules: (1) graph isomorphism encoders to extract domain-specific embeddings, (2) a progressive feature alignment module to gradually align source and target graph representations via KL divergence, and (3) an unsupervised infomax regularizer that maximizes mutual information to enhance feature quality. The model uses a 5-layer graph isomorphism network (GIN) encoder (128 hidden units), is trained with the Adam optimizer (learning rate $1\times10^{-3}$) for 200 epochs, and balances alignment and infomax losses with weights of 1.0 and 0.1, respectively.

\textbf{LG-DADA} \cite{bessadok2021brain}: LG-DADA (Learning-Guided Dual Adversarial Domain Alignment) addresses domain shift by generating source-to-target brain networks via dual adversarial training. It incorporates a manifold-based clustering module and two discriminators to simultaneously align domain-level and graph-level distributions. The model uses a two-layer graph autoencoder with 128-dimensional embeddings, five clusters for source subgroups, and loss weights set as $\lambda_{\text{domain}} = 1.0$, $\lambda_{\text{graph}} = 0.5$, and $\lambda_{\text{cluster}} = 0.2$. Training is conducted with Adam at a learning rate of $1 \times 10^{-4}$.


UFA-Net \cite{fang2023unsupervised} follows a domain-adversarial framework similar to DANN, using gradient reversal to encourage domain-invariant representations. The model employs multilayer perceptrons with ReLU activations as the feature generator and classifier, and is trained using Adam with a learning rate of 0.0001 and a batch size of 16.

In contrast to existing methods, the proposed UJA framework performs joint alignment of both marginal (domain-wise alignment) and conditional (class-wise alignment) distributions in an unsupervised setting. It incorporates a multi-head self-attention (MHSA) module to extract expressive node-level features from FC matrices, followed by adversarial training with a domain discriminator to enforce domain-wise alignment. To achieve class-wise alignment, two task-specific classifiers are introduced, and the prediction discrepancy between them on target samples is minimized using the sliced Wasserstein distance.

This unified alignment strategy enables UJA to deliver superior performance, particularly in challenging multi-site scenarios characterized by substantial inter-site distribution shifts. Empirical results demonstrate that UJA not only improves target-domain classification accuracy but also preserves semantic consistency and enhances generalization across heterogeneous domains.




\subsubsection{Effect of Feature Dimensionality}

To investigate the influence of representation dimensionality on classification performance, we varied the output dimension of the feature generator in the proposed UJA framework, while keeping all other components fixed. Specifically, we evaluate five configurations: 64, 128, 256, 512, and 1024.

As shown in Figure~\ref{fig:featuresize}, both ACC and AUC improve as the feature dimension increases up to 512. Beyond this point, performance saturates or slightly deteriorates, likely due to overfitting or the inclusion of redundant information. Lower-dimensional representations (e.g., 64 or 128) appear insufficient to capture the complexity of brain functional connectivity, limiting classification effectiveness.

These results suggest that a moderate feature dimensionality—specifically, 512—strikes an optimal balance between expressive capacity and generalization, yielding the best cross-domain performance.

\begin{figure}[h]
	\centering
	\includegraphics[width=0.75\linewidth]{./performance_vs_feature_size}
	\caption{Effect of feature dimensionality on model performance. The figure illustrates the relationship between the output dimension of the feature generator and classification performance in terms of ACC and AUC. Both metrics improve as the feature dimension increases up to 512, after which the performance saturates or slightly declines, indicating potential overfitting at very high dimensions.}
	\label{fig:featuresize}
\end{figure}

\subsubsection{Effect of Brain Atlas Selection}

To assess the robustness and generalizability of the proposed UJA model across different brain parcellation schemes, we conducted experiments using four representative atlases: AAL (116 ROIs), Harvard–Oxford (HO), Dosenbach (160 ROIs), and CC200. Each atlas partitions the brain into a different number and configuration of ROIs, resulting in distinct functional connectivity structures. Such variation poses non-trivial challenges for model robustness, as inconsistent spatial definitions may exacerbate cross-site variability.

We replicated the main cross-domain experiment under each atlas setting while keeping all other factors unchanged. As shown in Table~\ref{tab:atlas_comparison}, UJA achieved its best performance with the CC200 atlas, reaching an ACC of 69.53\%, an AUC of 71.33\%, and a favorable balance between sensitivity and specificity. Results with the other atlases, although slightly lower, remained competitive and consistent. For example, AAL and Dosenbach yielded comparable overall accuracy (66.05\% and 67.25\%, respectively), indicating that UJA performs stably across both anatomically and functionally defined parcellation schemes.

Overall, these results show that UJA performs robustly across different atlas types, with CC200 providing the best balance of accuracy, discrimination, and sensitivity–specificity. Such robustness to parcellation choice is crucial for practical use, as no single protocol is universally adopted in neuroimaging, underscoring the adaptability and applicability of the proposed method.

\begin{table}[htbp]
	\centering
	\caption{Classification performance under different brain atlases (sorted by ACC)}
	\label{tab:atlas_comparison}
	\begin{tabular}{lcccc}
		\toprule
		Atlas & ACC (\%) & AUC (\%) & SEN (\%) & SPE (\%) \\
		\midrule
		HO          & 64.82 & 66.10 & 65.40 & 64.24 \\
		AAL         & 66.05 & 67.35 & 67.20 & 64.90 \\
		Dosenbach   & 67.25 & 69.40 & 68.50 & 66.00 \\
		CC200       & \textbf{69.53} & \textbf{71.33} & \textbf{67.09} & \textbf{73.47} \\
		\bottomrule
	\end{tabular}
\end{table}
