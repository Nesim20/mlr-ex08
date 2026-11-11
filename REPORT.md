What I tried
I aimed to improve the robustness of the custom LeafNetBN pipeline (from Task 3) by introducing colour augmentation. I instructed the AI agent to add transforms.ColorJitter(brightness=0.2, contrast=0.2, saturation=0.2, hue=0.1) to the training transform pipeline. To simulate real-world usage on a phone camera, the agent also generated a "camera-like" validation set (the standard validation set modified with random crops, rotations, and colour jitter, frozen with a fixed seed). Both models were trained for 15 epochs to ensure a fair comparison.

The Results (15 epochs, LeafNetBN)

No colour augmentation (baseline)

Validation (last): 0.586

Validation (best): 0.690

Camera-like (last): 0.483

Camera-like (best): 0.517

With ColorJitter

Validation (last): 0.759

Validation (best): 0.759

Camera-like (last): 0.483

Camera-like (best): 0.517

What the changes brought
Validation Accuracy: Adding ColorJitter improved the standard validation accuracy, jumping from 0.586 to 0.759 (a gain of 5 correctly classified images out of 29). This difference is well above the one-image noise threshold, showing a strong regularisation and generalisation effect. It also eliminated the epoch-to-epoch swinging seen in the baseline.
Camera-like Accuracy (What didn't help): Surprisingly, the augmented model showed no improvement on the camera-like validation set. Both models peaked at a poor 0.517 (15 out of 29 images). This indicates that while ColorJitter helps the network learn the base images better, it does not buy the expected camera robustness.

Final Choice for the Phone
Based on these results, my custom LeafNetBN pipeline, even with geometric and colour augmentation, struggles with the complex variations of a live phone camera (peaking at ~51% accuracy). Therefore, for the actual phone deployment, I would stick with the pretrained MobileNetV3 (Task 2). The pretrained features from ImageNet are vastly superior in handling new environments and lighting conditions compared to a small network trained from scratch on a few dozen photos.