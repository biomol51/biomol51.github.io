+++
date = '2025-03-15T19:39:17+09:00'
draft = false
title = '若手ポスター発表賞審査フォーム'
+++

<style>
    input {
        width: 100%;
        padding: 0px;
        margin: 5px 0;
        flex-grow: 1;
    }
    label {
        display: block;
        font-weight: bold;
    }
    textarea {
        resize: vertical;
        width: 100%;
        min-height: 100px;
        height: 200px;
        padding: 0px;
        margin: 5px 0;
        flex-grow: 1;
    }
    .radio-inline, .label-inline {
        display: inline-block;
        margin-right: 10px;
    }
    th, td {
        text-align: center;
        vertical-align: middle!important;
        width: 20%;
    }
</style>

<div class="col-md-12" style="display: none" id="shinsa">
    <div class="lead" style="line-height: 2">
        <p>審査員：<span id="reviewer_name"></span> 先生</p>
        <p>審査コード： <span id="access_code"></span></p>
        <p>採点基準： 
            <ul>
                <li>ポスター：ポスターの見やすさ、分かりやすさ（10点満点）</li>
                <li>説明：明瞭に話しているか、全体の論理構成が適切かどうか（10点満点）</li>
                <li>質疑：質疑を的確に理解し、論理的に回答しているか（10点満点）</li>
            </ul>
        </p>
        <p class="text-primary">※審査終了後、 17:40 までに受付に提出してください<br>※絶対評価で採点を行ってください。基準として平均的なポスター発表の各項目の点数を5点 としてください。</p>
    </div>
    <div id="posters"></div>
    <p><button type="submit" class="btn btn-primary text-large">提出</button></p>
</div>

<script>
    window.onload = function() {
        let userInput = prompt("アクセスコードを入力してください：");
        fetch('https://onodalab.ees.hokudai.ac.jp/biomol51/review/?access_code=' + userInput, {
            method: 'GET',
            headers: {
                'Accept': 'application/json',
                'Access-Control-Allow-Origin': '*'
            }
        })
        .then(response => response.json())
        .then(data => {
            if ('error' in data) {
                alert('アクセスコードが間違っています。再度入力してください。');
                location.reload();
            }

            document.getElementById('shinsa').style.display = 'block';
            document.getElementById('reviewer_name').innerText = data.reviewer_name;
            document.getElementById('access_code').innerText = data.access_code;
            document.getElementById('posters').innerHTML = data.posters;

            document.querySelector('.btn-primary').addEventListener('click', submitReview);
        });
    };

    function submitReview() {
        // check if all scores are filled
        let scores = document.querySelectorAll('.score1, .score2, .score3');
        for (let i = 0; i < scores.length; i++) {
            let value = scores[i].value.trim();
            let num = parseInt(value);
            if ( value === '' || isNaN(num) || num < 0 || num > 10) {
                alert('全ての項目に0から10の整数で採点してください。');
                scores[i].focus();
                return;
            }
        }
        let reviewData = {
            'access_code': document.getElementById('access_code').innerText,
            'poster_id': $('.poster_id').map((_, el) => $(el).text().trim()).get(),
            'score1': $('.score1').map((_, el) => $(el).val() || '').get(),
            'score2': $('.score2').map((_, el) => $(el).val() || '').get(),
            'score3': $('.score3').map((_, el) => $(el).val() || '').get(),
            'comment': $('.comment').map((_, el) => $(el).text().trim()).get(),
        };
        fetch('https://onodalab.ees.hokudai.ac.jp/biomol51/submit/', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Access-Control-Allow-Origin': '*'
            },
            body: JSON.stringify(reviewData)
        })
        .then(response => response.json())
        .then(data => {
            if ('success' in data) {
                alert('審査結果を提出しました。ご協力いただき、ありがとうございます。');
            } else {
                alert('審査結果の提出に失敗しました。ページを更新して再試行してください。何度も失敗する場合は、事務局までお問い合わせください。');
            }
            location.href = '/';
        });
    }
</script>