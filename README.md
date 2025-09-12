Максим:
// Program.cs
// Универсальный тестовый инструмент (C#)
// Поддерживает: power (a^n), passwd (проверка пароля), gen_tests (генерация граничных тестов)
using System;
using System.Collections.Generic;
using System.Globalization;
using System.Linq;
using System.Numerics;
using System.Text;
class Program
{
    static void Main()
    {
        Console.OutputEncoding = Encoding.UTF8;
        ShowIntro();
        while (true)
        {
            Console.Write("Команда (power/passwd/gen_tests/0): ");
            var cmd = (Console.ReadLine() ?? "").Trim().ToLowerInvariant();
            if (cmd == "0"  cmd == "exit"  cmd == "quit") { Console.WriteLine("Выход."); break; }

            if (cmd == "power")
            {
                Console.Write("Введите a (число): ");
                var aStr = Console.ReadLine() ?? "";
                Console.Write("Введите n (целое): ");
                var nStr = Console.ReadLine() ?? "";
                Console.WriteLine(PowerCommand(aStr, nStr));
            }
            else if (cmd == "passwd")
            {
                Console.Write("Введите пароль для проверки: ");
                string pwd = Console.ReadLine() ?? "";
                var r = CheckPassword(pwd);
                Console.WriteLine($"Результат: {(r.Ok ? "OK" : "FAIL")}");
                if (r.Errors.Count > 0) Console.WriteLine("Ошибки: " + string.Join("; ", r.Errors));
                Console.WriteLine($"Оценка силы: {r.Strength} (баллов: {r.Score})");
            }
            else if (cmd == "gen_tests" || cmd == "gen_tests_for_range")
            {
                Console.Write("min value: ");
                var mn = Console.ReadLine() ?? "";
                Console.Write("max value: ");
                var mx = Console.ReadLine() ?? "";
                var vals = GenTestsForRange(mn, mx);
                Console.WriteLine("Сгенерированные тест-значения:");
                if (vals is string)
                {
                    Console.WriteLine(vals);
                }
                else if (vals is List<string> list)
                {
                    foreach (var v in list) Console.WriteLine("  " + v);
                }
            }
            else
            {
                Console.WriteLine("Неизвестная команда. Повторите.");
            }
        }
    }

    static void ShowIntro()
    {
        Console.WriteLine("Универсальный тестовый инструмент");
        Console.WriteLine("Меню:");
        Console.WriteLine(" 1) power    - вычислить a^n (a - число, n - целое)");
        Console.WriteLine(" 2) passwd   - проверить пароль по простым правилам");
        Console.WriteLine(" 3) gen_tests- сгенерировать минимальный набор граничных тестов для одного числового параметра");
        Console.WriteLine(" 0) exit");
        Console.WriteLine();
    }

    // ----- power -----
    static string PowerCommand(string aStr, string nStr)
    {
        if (!double.TryParse(aStr.Replace(',', '.'), NumberStyles.Any, CultureInfo.InvariantCulture, out double a))
            return "Ошибка: a должно быть числом.";

        if (!long.TryParse(nStr, NumberStyles.Any, CultureInfo.InvariantCulture, out long n))
            return "Ошибка: n должно быть целым числом.";

        if (a == 0.0 && n < 0)
            return "Ошибка: 0 в отрицательной степени не определено.";

        // Попробуем аккуратно вычислить целую степень для целых оснований с помощью BigInteger, если возможно
        if (IsIntegerString(aStr) && Math.Abs(a) <= 1e18 && Math.Abs(n) <= 10000)
        {
            try
            {
                BigInteger baseInt = BigInteger.Parse(aStr, CultureInfo.InvariantCulture);
                if (n >= 0)
                {
                    BigInteger bigRes = BigInteger.Pow(baseInt, (int)n);
                    return $"{baseInt} ** {n} = {bigRes}";

}
                else
                {
                    // отрицательная степень: результат дробный
                    double val = Math.Pow((double)baseInt, (double)n);
                    return $"{baseInt} ** {n} = {val} (double)";
                }
            }
            catch
            {
                // fallthrough to double pow
            }
        }

        double res = Math.Pow(a, (double)n);
        // Если результат фактически целый — показать как целое
        if (Math.Abs(res - Math.Round(res)) < 1e-12 && Math.Abs(res) <= (double)long.MaxValue)
            return $"{a} ** {n} = {(long)Math.Round(res)}";
        else
            return $"{a} ** {n} = {res}";
    }

    static bool IsIntegerString(string s)
    {
        s = s?.Trim();
        if (string.IsNullOrEmpty(s)) return false;
        if (s.StartsWith("+") || s.StartsWith("-")) s = s.Substring(1);
        return s.All(char.IsDigit);
    }

    // ----- password check -----
    class PwdResult
    {
        public bool Ok { get; set; } = true;
        public List<string> Errors { get; set; } = new List<string>();
        public int Score { get; set; } = 0;
        public string Strength { get; set; } = "очень слабый";
    }

    static PwdResult CheckPassword(string pwd,
                                   int minLen = 8, int maxLen = 30,
                                   bool requireDigit = true,
                                   bool requireLetter = true,
                                   bool allowCyrillic = false)
    {
        var r = new PwdResult();
        int L = pwd.Length;
        if (L < minLen) { r.Ok = false; r.Errors.Add($"Длина {L} < {minLen}"); }
        if (L > maxLen) { r.Ok = false; r.Errors.Add($"Длина {L} > {maxLen}"); }
        if (requireDigit && !pwd.Any(char.IsDigit)) { r.Ok = false; r.Errors.Add("Не содержит цифры"); }
        if (requireLetter && !pwd.Any(char.IsLetter)) { r.Ok = false; r.Errors.Add("Не содержит букв"); }
        if (!allowCyrillic && ContainsCyrillic(pwd)) { r.Ok = false; r.Errors.Add("Содержит кириллические символы (запрещены)"); }

        int score = 0;
        if (L >= 12) score++;
        if (pwd.Any(char.IsLower) && pwd.Any(char.IsUpper)) score++;
        if (pwd.Any(char.IsDigit)) score++;
        if (pwd.Any(ch => char.IsPunctuation(ch) || char.IsSymbol(ch))) score++;
        r.Score = score;
        r.Strength = score switch
        {
            0 => "очень слабый",
            1 => "слабый",
            2 => "средний",
            3 => "сильный",
            _ => "очень сильный"
        };
        return r;
    }

    static bool ContainsCyrillic(string s)
    {
        foreach (char ch in s)
        {
            // кириллический диапазон
            if (ch >= '\u0400' && ch <= '\u04FF') return true;
            if (ch >= '\u0500' && ch <= '\u052F') return true;
            if (ch >= '\u2DE0' && ch <= '\u2DFF') return true;
            if (ch >= '\uA640' && ch <= '\uA69F') return true;
        }
        return false;
    }

    // ----- gen_tests -----
    static object GenTestsForRange(string minStr, string maxStr)
    {
        if (string.IsNullOrWhiteSpace(minStr) || string.IsNullOrWhiteSpace(maxStr))
            return "Ошибка: min или max пусты.";

        // Попытаемся распознать как целые (без точки/е), иначе как double
        bool minLooksFloat = minStr.Contains('.') || minStr.IndexOf('e', StringComparison.OrdinalIgnoreCase) >= 0;
        bool maxLooksFloat = maxStr.Contains('.') || maxStr.IndexOf('e', StringComparison.OrdinalIgnoreCase) >= 0;
        if (!minLooksFloat && !maxLooksFloat)
        {
            // integer path
            if (!long.TryParse(minStr, NumberStyles.Any, CultureInfo.InvariantCulture, out long a) ||
                !long.TryParse(maxStr, NumberStyles.Any, CultureInfo.InvariantCulture, out long b))
                return "Ошибка: не удалось распарсить целые значения.";

long mid = a + (b - a) / 2;
            var vals = new List<string>
            {
                (a - 1).ToString(),
                a.ToString(),
                (a + 1).ToString(),
                mid.ToString(),
                (b - 1).ToString(),
                b.ToString(),
                (b + 1).ToString()
            };
            return vals;
        }
        else
        {
            // float path
            if (!double.TryParse(minStr.Replace(',', '.'), NumberStyles.Any, CultureInfo.InvariantCulture, out double a) ||
                !double.TryParse(maxStr.Replace(',', '.'), NumberStyles.Any, CultureInfo.InvariantCulture, out double b))
                return "Ошибка: не удалось распарсить вещественные значения.";

            double mid = (a + b) / 2.0;
            double eps = Math.Max(1e-6, Math.Abs(b - a) * 1e-6);
            var vals = new List<string>
            {
                FormatDouble(a - eps),
                FormatDouble(a),
                FormatDouble(a + eps),
                FormatDouble(mid),
                FormatDouble(b - eps),
                FormatDouble(b),
                FormatDouble(b + eps)
            };
            return vals;
        }
    }

    static string FormatDouble(double v)
    {
        // Убрать лишние нули, использовать InvariantCulture
        return v.ToString("G17", CultureInfo.InvariantCulture);
    }
}
