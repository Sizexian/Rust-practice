# 基于官方出题练习对应的代码
## 链接地址：https://kaisery.github.io/trpl-zh-cn/ch08-03-hash-maps.html

***

#### 给定一系列数字，使用 vector 并返回这个列表的中位数（排列数组后位于中间的值）和众数（mode，出现次数最多的值；这里哈希 map 会很有帮助）。
```
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
```
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

#### AI给出的答案
```
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