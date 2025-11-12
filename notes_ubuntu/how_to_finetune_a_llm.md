数据准备

```python
class textdataset(dataset):
    def __init__(self, file_path,tokenizer,block_size=512):
        # 读取文件
        # 按照某种规则进行分割
        # 对文本进行编码
    def __len__(self):
        return len(self.examples)
    
    def __getitem__(self,idx):
			return self.examples[idx]
        
def get_batch(tokenizer, train_file, val_file=None,block_size):
    # 准备训练集和验证集
    return train_dataset, val_dataset
```

---

模型加载和配置

```python
def setup_model_and_tokenizer(model_name="gpt2", output_dir="./fine-tuned-model):
	# 加载预训练模型
	return model, tokenizer, output_dir
```

---

训练参数设置

```python
def get_training_args(output_dir, btach_size=4, grad_accum=4):
	# 设置训练参数pythonpython
    effective_batch_size = batch_size * grad_accum
    
    training_args = TrainingArguments(
        output_dir=output_dir,
        overwrite_output_dir=True,
        num_train_epochs=3,
        per_device_train_batch_size=batch_size,
        per_device_eval_batch_size=batch_size,
        gradient_accumulation_steps=grad_accum,
        warmup_steps=100,
        logging_steps=10,
        save_steps=500,
        eval_steps=500,
        evaluation_strategy="steps",
        save_strategy="steps",
        load_best_model_at_end=True,
        learning_rate=5e-5,
        weight_decay=0.01,
        fp16=torch.cuda.is_available(),
        dataloader_pin_memory=False,
        report_to=None,  # 禁用wandb等记录
    )
    
    return training_args
```

---

完整训练流程

```python
def fine_tune_model(train_file, val_file=None, model_name="gpt2"):
    """完整的微调流程"""
    
    # 1. 设置模型和tokenizer
    print("🔄 加载模型和tokenizer...")
    model, tokenizer, output_dir = setup_model_and_tokenizer(model_name)
    
    # 2. 准备数据
    print("📚 准备数据...")
    train_dataset, val_dataset = prepare_data(tokenizerpython, train_file, val_file)
    
    print(f"训练样本数: {len(train_dataset)}")
    print(f"验证样本数: {len(val_dataset)}")
    
    # 3. 设置数据收集器
    data_collator = DataCollatorForLanguageModeling(
        tokenizer=tokenizer,
        mlm=False,  # 对于GPT是因果语言模型，不是掩码语言模型
    )
    
    # 4. 设置训练参数
    training_args = get_training_args(output_dir)
    
    # 5. 创建Trainer
    trainer = Trainer(
        model=model,
        args=training_args,
        data_collator=data_collator,
        train_dataset=train_dataset,
        eval_dataset=val_dataset,
        tokenizer=tokenizer,
    )
    
    # 6. 开始训练
    print("🚀 开始训练...")
    trainer.train()
    
    # 7. 保存最终模型
    print("💾 保存模型...")
    trainer.save_model()
    tokenizer.save_pretrained(output_dir)
    
    print(f"✅ 训练完成！模型保存在: {output_dir}")
    
    return model, tokenizer
    python
```

---

推理和使用

```python
def generate_text(model, tokenizer, prompt, max_length=100):
    """使用微调后的模型生成文本"""
    
    inputs = tokenizer(prompt, return_tensors="pt")
    
    with torch.no_grad():
        outputs = model.generate(
            inputs.input_ids,
            max_length=max_length,
            num_return_sequences=1,
            temperature=0.8,
            do_sample=True,
            pad_token_id=tokenizer.eos_token_id,
        )
    
    generated_text = tokenizer.decode(outputs[0], skip_special_tokens=True)
    return generated_text

def load_fine_tuned_model(model_path):
    """加载微调后的模型"""
    tokenizer = AutoTokenizer.from_pretrained(model_path)
    model = AutoModelForCausalLM.from_pretrained(model_path)
    return model, tokenizer
```

---

主函数

```python
def main():
    """主函数"""
    
    # 配置参数
    config = {
        "train_file": "your_data.txt",  # 你的txt文件路径
        "val_file": None,               # 验证集文件（可选）
        "model_name": "gpt2",           # 预训练模型名称
        "output_dir": "./my-fine-tuned-model",
    }
    
    # 检查数据文件
    if not os.path.exists(config["train_file"]):
        print(f"❌ 训练文件不存在: {config['train_file']}")
        return
    
    try:
        # 执行微调
        model, tokenizer = fine_tune_model(
            train_file=config["train_file"],
            val_file=config["val_file"],
            model_name=config["model_name"]
        )
        
        # 测试生成
        print("\n🧪 测试生成...")
        test_prompt = "今天天气很好，"
        generated = generate_text(model, tokenizer, test_prompt)
        print(f"输入: {test_prompt}")
        print(f"生成: {generated}")
        
    except Exception as e:
        print(f"❌ 训练过程中出错: {e}")
        import traceback
        traceback.print_exc()

if __name__ == "__main__":
    main()
```

