# 基于官方出题练习对应的代码
## 链接地址：https://kaisery.github.io/trpl-zh-cn/ch08-03-hash-maps.html

***

#### 给定一系列数字，使用 vector 并返回这个列表的中位数（排列数组后位于中间的值）和众数（mode，出现次数最多的值；这里哈希 map 会很有帮助）。
```rust
use std::collections::HashMap;

use rand::Rng;

fn main(){
    let mut v:Vec<i32>=Vec::new();
    {
        let mut rng=rand::thread_rng();
        for _ in 1..100 {
            v.push(rng.gen_range(1..20));
        }
    }
    println!("{:?}",&v);//输出生成的100个数
    let mut map:HashMap<i32, i32>=HashMap::new();
    {
        for num in v {
            let count=map.entry(num).or_insert(0);
            *count+=1;
        }
    }
    println!("{:?}",map);//输出统计的数字及出现次数
    {
        let max_by_key=map.iter().max_by_key(|entry | entry.1).unwrap();
        println!("{:?}",max_by_key);
    }
    {//中位数
        let mut v=Vec::new();
        for (num,_) in map {
            v.push(num);
        }
        v.sort();
        println!("{:?}",v);
        let len=v.len();
        if len%2!=0 {
            println!("{}",v[len/2]);
        }
        else {
            println!("{}",(v[len/2]+v[len/2+1]) as f32/2.0);
        }
    }
}
```

***

#### 将字符串转换为 Pig Latin，也就是每一个单词的第一个辅音字母被移动到单词的结尾并增加 “ay”，所以 “first” 会变成 “irst-fay”。元音字母开头的单词则在结尾增加 “hay”（“apple” 会变成 “apple-hay”）。牢记 UTF-8 编码！
```rust
use std::{io::Write, process::exit};

fn main(){
    print!("Input:");
    std::io::stdout().flush().expect("flush error");
    let mut s: String=String::new();
    std::io::stdin().read_line(&mut s).expect("input error");
    // println!("{:?}",s);
    let s=s.trim();//之前没加trim导致字符串里有\r\n一直换行
    // println!("{:?}",s);
    let alphabet:Vec<char>="abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ".chars().collect();
    if !s.chars().all(|c| alphabet.contains(&c)) {
        println!("Only letters are accepted");
        exit(-1);
    }
    let v: Vec<char>=vec!['a','e','i','o','u'];
    {
        let first_letter: Option<char>=s.chars().next();
        let mut result=String::new();
        match first_letter {
            Some(letter) => {
                if v.contains(&letter) {
                    result.insert_str(result.len(), format!("{}-hay",s).as_str());
                    println!("{result}");
                }
                else {//由于不能确定size，所以不能用切片。不知道有更方便的方法结果写了一坨屎出来
                    let mut first_iteration=true;
                    for c in s.chars(){
                        if first_iteration {
                            first_iteration=false;
                            continue;
                        }
                        result.insert(result.len(), c);
                    }
                    result.insert_str(result.len(), format!("-{}ay",letter).as_str());
                    println!("{result}");
                }
            },
            None => println!("empty input!")
        }
    }
}
```

#### AI对代码检查做出的改进
Based on the code you provided, it appears that you are trying to convert a string to Pig Latin. However, there are a few issues with your code that need to be addressed. Let's go through them step by step:

1. **Input Handling**: Your code prompts the user for input and reads it from the standard input. This part seems fine.

2. **Trimming the Input**: You correctly use the `trim()` method to remove any leading or trailing whitespace from the input string.

3. **Alphabet Check**: Your code checks if the input string contains only letters. This is a good validation step. However, the implementation can be improved. Instead of manually creating a vector of all alphabets, you can use the `is_alphabetic()` method of the `char` type to check if each character in the string is a letter.

4. **Pig Latin Conversion**: Your code attempts to convert the input string to Pig Latin. However, there are a few issues with the implementation:

   - **Handling Vowels**: Your code correctly checks if the first letter of the word is a vowel. If it is, you append "-hay" to the end of the word. However, you are not handling the case where the word starts with an uppercase vowel. You need to convert the first letter to lowercase before checking if it is a vowel.

   - **Handling Consonants**: Your code attempts to handle words that start with consonants. However, the implementation is incorrect. You are iterating over the characters of the input string and appending them to the result string, excluding the first character. This approach will not give you the desired Pig Latin conversion. Instead, you should move the first consonant to the end of the word and append "-ay". There are multiple ways to achieve this, but one simple approach is to use the `split_at()` method to split the string into two parts: the first character and the rest of the string. Then, you can concatenate them in the desired order.

5. **Printing the Result**: Your code uses `println!()` to print the result. However, you are not correctly formatting the output string. The variable `result` should be enclosed in curly braces `{}` to be properly interpolated in the string.

Here's an updated version of your code with the necessary corrections:

```rust
use std::{io::Write, process::exit};

fn main() {
    print!("Input: ");
    std::io::stdout().flush().expect("flush error");
    let mut s = String::new();
    std::io::stdin().read_line(&mut s).expect("input error");
    let s = s.trim();

    if !s.chars().all(|c| c.is_alphabetic()) {
        println!("Only letters are accepted");
        exit(-1);
    }

    let v: Vec<char> = vec!['a', 'e', 'i', 'o', 'u'];

    if let Some(first_letter) = s.chars().next() {
        let mut result = String::new();

        if v.contains(&first_letter.to_ascii_lowercase()) {
            result = format!("{}-hay", s);
        } else {
            let (first, rest) = s.split_at(1);
            result = format!("{}-{}ay", rest, first);
        }

        println!("{}", result);
    } else {
        println!("Empty input!");
    }
}
```
总结：ai似乎并没有完全理解我写的代码，经过实际测试，辅音字母开头可以完成转换。但ai写的使用了更多rust原有的方法。思路更清晰更加方便快捷。

#### AI给出的答案
```rust
fn convert_to_pig_latin(input: &str) -> String {
    let vowels = ['a', 'e', 'i', 'o', 'u'];
    let mut pig_latin_words = Vec::new();

    for word in input.split_whitespace() {
        let mut chars = word.chars();
        if let Some(first_char) = chars.next() {
            if vowels.contains(&first_char.to_ascii_lowercase()) {
                let pig_latin_word = format!("{}-hay", word);
                pig_latin_words.push(pig_latin_word);
            } else {
                let remaining_word: String = chars.collect();
                let pig_latin_word = format!("{}-{}ay", remaining_word, first_char);
                pig_latin_words.push(pig_latin_word);
            }
        }
    }

    pig_latin_words.join(" ")
}

fn main() {
    let input = "Convert strings to pig latin";
    let pig_latin = convert_to_pig_latin(input);
    println!("{}", pig_latin);
}
```


***

#### 使用哈希 map 和 vector，创建一个文本接口来允许用户向公司的部门中增加员工的名字。例如，“Add Sally to Engineering” 或 “Add Amir to Sales”。接着让用户获取一个部门的所有员工的列表，或者公司每个部门的所有员工按照字典序排列的列表。
```rust
use std::{collections::HashMap, process::exit};

fn main() {
    println!("Input 0 to exit");
    let mut map: HashMap<String, String> = HashMap::new(); // Change the type to hold owned String values
    loop {
        let mut s = String::new();
        std::io::stdin().read_line(&mut s).expect("input error");
        let s_trimmed = s.trim();
        if s_trimmed == "0" {
            break;
        }

        if !s_trimmed.chars().all(|c| c.is_alphabetic()||c==' ') {
            println!("Only letters are accepted");
            exit(-1);
        }

        let words: Vec<&str> = s_trimmed.split_whitespace().collect();
        map.insert(words[1].to_string(), words[3].to_string()); // Convert the references to owned String values
        print!("{:?}",words);
    }
    println!("{:?}", map);
}
```