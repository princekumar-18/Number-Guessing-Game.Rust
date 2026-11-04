// Number Guessing Challenge
// Created by: Prince

use std::io::{self, Write};
use std::time::{SystemTime, UNIX_EPOCH};

fn read_int(prompt: &str) -> i64 {
    loop {
        print!("{prompt}");
        io::stdout().flush().unwrap();

        let mut line = String::new();
        if io::stdin().read_line(&mut line).is_err() {
            continue;
        }

        match line.trim().parse::<i64>() {
            Ok(value) => return value,
            Err(_) => println!("Please enter a whole number."),
        }
    }
}

fn random_inclusive(low: i64, high: i64) -> i64 {
    // Small standard-library pseudo-random generator.
    let time = SystemTime::now()
        .duration_since(UNIX_EPOCH)
        .unwrap()
        .as_nanos() as u64;

    let mut x = time ^ (low as u64).rotate_left(17) ^ (high as u64).rotate_left(31);
    x ^= x << 13;
    x ^= x >> 7;
    x ^= x << 17;

    low + (x % ((high - low + 1) as u64)) as i64
}

fn main() {
    println!("=== Number Guessing Challenge ===");

    loop {
        println!("\nDifficulty: 1) Easy  2) Medium  3) Hard  4) Custom");
        let choice = read_int("> ");

        let (mut low, mut high, mut attempts) = match choice {
            1 => (1, 50, 10),
            3 => (1, 500, 10),
            4 => {
                let low = read_int("Lowest possible number: ");
                let high = loop {
                    let value = read_int("Highest possible number: ");
                    if value > low {
                        break value;
                    }
                    println!("Must be greater than the lowest number.");
                };
                let attempts = loop {
                    let value = read_int("Number of guesses (1-100): ");
                    if (1..=100).contains(&value) {
                        break value;
                    }
                    println!("Choose between 1 and 100.");
                };
                (low, high, attempts)
            }
            _ => (1, 100, 8),
        };

        // Keep the values mutable-free after setup.
        let secret = random_inclusive(low, high);
        let mut history: Vec<i64> = Vec::new();
        let mut won = false;

        println!("Guess from {low} to {high}. You have {attempts} guesses.");

        let mut turn = 1;
        while turn <= attempts {
            let guess = read_int(&format!("Guess {turn}/{attempts} (-9999 to quit): "));

            if guess == -9999 {
                break;
            }
            if guess < low || guess > high {
                println!("Stay within the range.");
                continue;
            }
            if history.contains(&guess) {
                println!("You already tried that number.");
                continue;
            }

            history.push(guess);

            if guess == secret {
                println!("Correct! You won in {turn} guesses.");
                won = true;
                break;
            }

            let distance = (secret - guess).abs();
            let temperature = if distance <= 3 {
                "very hot"
            } else if distance <= 10 {
                "hot"
            } else {
                "cold"
            };
            let direction = if guess < secret { "Too low" } else { "Too high" };
            println!("{direction} and {temperature}.");

            turn += 1;
        }

        if !won {
            println!("The number was {secret}.");
        }

        print!("Play again? (y/n): ");
        io::stdout().flush().unwrap();
        let mut answer = String::new();
        io::stdin().read_line(&mut answer).unwrap();

        if !answer.trim().eq_ignore_ascii_case("y") {
            break;
        }
    }

    println!("Thanks for playing!");
}
