## Contributions

- First application of Transformer-based models in 3D medical image segmentation
- Deformable self-attention mechanism for less complexity and multi-scale features

## Previous approaches

### TransUnet

CNN and Transformer are cascaded to produce an encoder, but optimization is challenging. The author locates finds self-attention problematic.

- long training time to focus the attention (examining each pixel in 3D images is impractical)
- a vanilla transformer is not capable of processing multi-scale and high-esolution feature maps (computational complexity)

CoTr proposes **deformable self attention mechanism** to address these issues.

MS-DMSA layer - focuses only on small set of key sampling locations

## De-Trans encoder

De-Trans encoder: composition of an input-to-sequence layer and LD stacked deformable Transformer (DeTrans) layers.

Multi-Scale DeforMable Self Attention (MS-DMSA) is used for long-range contextual encoding.

- First, flatten the feature maps of CNN encoder
- Then, supplement 3D positional encoding sequence to the flattened tensor.
- 


```python
# # %%%%%%%%%%%%% CoTr
x_convs = self.backbone(inputs)
x_fea, masks, x_posemb = self.posi_mask(x_convs)
x_trans = self.encoder_Detrans(x_fea, masks, x_posemb)

# # Single_scale
# # x = self.transposeconv_stage2(x_trans.transpose(-1, -2).view(x_convs[-1].shape))
# # skip2 = x_convs[-2]
# Multi-scale
# 맨밑 384x2x2
x = self.transposeconv_stage2(x_trans[:, x_fea[0].shape[-3]*x_fea[0].shape[-2]*x_fea[0].shape[-1]::].transpose(-1, -2).view(x_convs[-1].shape)) # x_trans length: 12*24*24+6*12*12=7776
skip2 = x_trans[:, 0:x_fea[0].shape[-3]*x_fea[0].shape[-2]*x_fea[0].shape[-1]].transpose(-1, -2).view(x_convs[-2].shape)

x = x + skip2
x = self.stage2_de(x)
ds2 = self.ds2_cls_conv(x)

# 그 위 192x2x2
x = self.transposeconv_stage1(x)
skip1 = x_convs[-3]
x = x + skip1
x = self.stage1_de(x)
ds1 = self.ds1_cls_conv(x)

# 맨위 64x2x2
x = self.transposeconv_stage0(x)
skip0 = x_convs[-4]
x = x + skip0
x = self.stage0_de(x)
ds0 = self.ds0_cls_conv(x)


result = self.upsamplex2(x)
result = self.cls_conv(result)

return [result, ds0, ds1, ds2]
```

