# Rust改善代码小技巧_之代码块
```rust
struct Message(usize);

fn get_msg(msg: &Option<Message>) -> String {
    let mut rep_msg = "Ok".to_string();
    if let Some(msg) = msg {
        rep_msg = format!("{}", msg.0)
    }
    rep_msg
}

fn get_msg_1(msg: &Option<Message>) -> String {
    let rep_msg = if let Some(msg) = msg {
        format!("{}", msg.0)
    } else {
        "Ok".to_string()
    };
    rep_msg
}

fn get_msg_2(msg: &Option<Message>) -> String {
    let rep_msg = {
        if let Some(msg) = msg {
            format!("{}", msg.0)
        } else {
            "Ok".to_string()
        }
    };
    rep_msg
}

fn get_msg_3(msg: &Option<Message>) -> String {
    msg.as_ref().map(|msg| format!("{}", msg.0)).unwrap_or("Ok".to_string())
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_get_msg() {
        let key = Some(Message(32usize));
        let expect = "32".to_string();
        assert_eq!(expect, get_msg(&key));
        assert_eq!(expect, get_msg_1(&key));
        assert_eq!(expect, get_msg_2(&key));
        assert_eq!(expect, get_msg_3(&key));
    }
}
```

//TODO add desc...

