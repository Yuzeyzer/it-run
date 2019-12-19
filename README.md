## Atabek Kurbanov
## contact:
##### *1 phone nubmer: +996773152695*
##### *2 email: kurbanof.atabek@gmail.com*
### Summary:
Dynamic and motivated professional with a proven record of generating and building relationships, managing projects from concept to completion, designing educational strategies, and coaching individuals to success. Skilled in building cross-functional teams, demonstrating exceptional communication skills, and making critical decisions during challenges. Adaptable and transformational leader with an ability to work independently, creating effective presentations, and developing opportunities that further establish organizational goals.

## Skills: 
1.Broad and extensive knowledge of the software development process and its technologies

2.Strong knowledge of computer languages, such as Java, C++, PHP, and SQL

3.Strong background in coding

4.Strong knowledge of user interfaces

5.Strong knowledge of HTML technologies and web frameworks

6.Experience with database creation and maintenance

## Code example:
char peek()
{
  char ch = EOF;
  if (cin >> ch)
    cin.unget();
  return ch;
}

// Вспомогательная функция для установки ошибки.
void error(const char *message)
{
  cerr << message;
  cin.setstate(ios::failbit);
}

// Числовой приоритет операции.
int precedence(char op)
{
  switch (op)
  {
  case '+': case '-': return 100;
  case '*': case '/': case '%': return 200;
  case '^': return 300;
  default: return 0;
  }
}

// Операция op связывает операнды слева направо?
bool is_left_associative(char op)
{
  return op != '^';
}


double infix()
{
  stack<char> ops;  // Стек операций.
  stack<double> xs; // Стек операндов.

  // Вложенная функция, выполняющая операцию на вершине стека ops.
  // Возвращает "успех" (false если произошла ошибка, true если выполнено успешно).
  auto do_next_op = [&]() -> bool
  {
    double x, y;
    if (xs.empty())
    {
      error("not enough operands\n");
      return false;
    }

    y = xs.top();
    xs.pop();

    if (xs.empty())
    {
      xs.push(y);
      error("not enough operands\n");
      return false;
    }

    x = xs.top();

    switch (ops.top())
    {
    case '+': x = x + y; break;
    case '-': x = x - y; break;
    case '*': x = x * y; break;
    case '/': x = x / y; break;
    case '%': x = fmod(x, y); break;
    case '^': x = pow(x, y); break;
    default:
      error("internal error\n");
      return false;
    }

    // Сохранить результат операции вместо операндов на стеке xs.
    xs.top() = x;
    // Убрать со стека операций выполненную операцию.
    ops.pop();
    return true;
  };


  bool awaiting_term = true; // Ожидает терм (true) или операцию (false)?
  while (true)
  {
    if (awaiting_term) // Ожидается терм.
    {
      double x;
      switch (char ch = peek())
      {
      // - и + могут стоять перед скобкой (, а не перед числом,
      // поэтому обрабатываем этот случай особо.
      case '-': case '+':
        if (cin.ignore() && cin.peek() == '(')
        {
          // Имитируем отрицание вычитанием из нуля.
          if (ch == '-')
          {
            xs.push(0);
            ops.push('-');
          }
          
          cin.ignore(); // Убрать открывающую скобку из потока.
          ops.push('(');
          break;
        }
        cin.unget(); // Не скобка -- возвращаем - или + обратно.
        // "Полноценный калькулятор" должен поддерживать унарные операции непосредственно.
        // Данный пример упрощён: поддерживаются только бинарные инфиксные операции.
      
      case '.':
      case '0': case '1': case '2': case '3': case '4':
      case '5': case '6': case '7': case '8': case '9':
        if (!(cin >> x))
        {
          error("number expected\n");
          goto Finish;
        }
        
        xs.push(x);
        awaiting_term = false;
        break;

      case '(':
        cin.ignore(); // Убрать открывающую скобку из потока.
        ops.push('(');
        break;

      case ')':
        error("unmatched ')' found\n");
        goto Finish;

      default:
        error("term expected, '");
        cerr << peek() << "' found\n";
        goto Finish;
      }
    }
    else // Ожидается операция.
    {
      switch (char op = peek())
      {
      case '+': case '-': case '*': case '/': case '%': case '^':
        // Основное отличие от предыдущего варианта калькулятора,
        // позволяющее учитывать приоритеты операций, находится здесь.
        {
          cin.ignore(); // Убрать знак операции с потока.
          const auto op_prec = precedence(op); // Приоритет считанной операции.
          const auto op_ltr = is_left_associative(op); // Левоассоциирующая операция?

          // Выполнить все предыдущие операции, имеющие приоритет выше op,
          // либо равный приоритет при условии, что op -- левоассоциирующая.
          while (!ops.empty() && ops.top() != '(')
          {
            const auto left_op_prec = precedence(ops.top());
            // Отрицание условия, записанного в комментарии выше (условие выхода из цикла).
            if ((op_ltr && left_op_prec < op_prec)
            ||  (!op_ltr && left_op_prec <= op_prec))
              break;
            
            if (!do_next_op())
              goto Finish;
          }

          // Положить на стек операций следующую операцию.
          ops.push(op);
          // Теперь ожидаем терм.
          awaiting_term = true;
        }
        break;

      case ')':
        // Убрать скобку с потока cin.
        cin.ignore();
        // Выполнять операции, пока не встретится открывающая скобка.
        while (true)
        {
          if (ops.empty())
          {
            error("unmatched ')' found\n");
            goto Finish;
          }

          if (ops.top() == '(')
          {
            ops.pop();
            break;
          }

          if (!do_next_op())
            goto Finish;
        }
        break;

      case EOF:
        // Выполнить все оставшиеся операции.
        while (!ops.empty())
        {
          if (ops.top() == '(')
          {
            error("unmatched '(' found\n");
            break;
          }

          if (!do_next_op())
            break;
        }
        goto Finish;

      default:
        error("operation expected: '");
        cerr << op << "'\n";
        goto Finish;
      }
    }
  }

Finish:
  if (xs.empty())
  {
    error("not enough operands\n");
    return 0.;
  }

  return xs.top();
}
## Опыт работы:
Разрабатывал приложение для детей с особенностями в развитии, в результате приложение стало востребовано, как для родителей «особых» детей, так и для терапевтов. Приложение состоит из мобильной части для пользователей и веб-сайта для администратора с возможностями контролирования пользователей.
Возможности приложения:
— запись видео сеансов, передача видеофайлов с приложения на веб-сайт;
— запись данных вручную внутри приложения: время проведения терапии, симптомы и их тяжесть;
— составление графиков анализа симптомов.

## Образование: 
Банковское Дело и страхование 
Front - end Developer

## Уровень английского:
Intermediate